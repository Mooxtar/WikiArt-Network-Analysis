# WikiArt Network Analysis (Artists · Movements · Schools · Institutions)

This project builds a **heterogeneous network** from WikiArt-style metadata and then uses **NetworkX** + **graph analytics** to answer questions like:

- Which **movements** appear most “central” (via shared artists)?
- Which **artists** look most “influential” in the artist→artist influence layer?
- Which **institutions / schools** act as hubs?
- Which **nationalities** dominate the dataset?
- What are the **largest communities** in the network?
- How can we **visualize** the graph (static + interactive)?

All analysis is in the notebook: **`WikiArt_network_analysis.ipynb`**.

---

## Data

The notebook expects these CSV files next to the notebook (or inside `data/` if you update the paths):

- `artists.csv` (≈ 2,996 rows)
- `relationships.csv` (≈ 2,996 rows)
- `institutions.csv` (≈ 73 rows)
- `schools.csv` (≈ 220 rows)

### What the files contain (high-level)

- **artists.csv**: artist identifier (`artisturl`), name/title, nationality, total works, year, image
- **relationships.csv**: list-like fields for graph links (e.g., friends, influenced_by, influenced_on, movements, school, institution)
- **institutions.csv / schools.csv**: catalog tables with titles and URLs

> Note: the notebook includes parsing utilities to convert stringified lists into Python lists safely, so the relationships table can be used as edge lists.

---

## Environment & Dependencies

Main libraries used:

- `pandas`, `numpy` (data handling)
- `networkx` (graph construction + metrics)
- `matplotlib` (static plots)
- `plotly` (interactive visualization)

Install:

```bash
pip install pandas numpy networkx matplotlib plotly
```

Run locally:

```bash
jupyter notebook WikiArt_network_analysis.ipynb
```

---

## Workflow (what the notebook does)

### Step 1 — Preprocessing & EDA
- Load the 4 datasets and check missingness.
- Parse “list-like” columns in `relationships.csv` (e.g., comma/JSON-like lists).
- Quick distributions (nationalities, movements, artworks count) to understand the long-tail structure.

### Step 2 — Build a heterogeneous NetworkX graph
- Create a **directed graph** `G` with typed nodes:
  - `artist` (key: `artisturl`)
  - `movement` (normalized ID)
  - `school` (`school::...`)
  - `institution` (`institution::...`)
- Add edges from relationships with **weights** (repeated relations increase weight) and labeled relation types:
  - `friend`
  - `influenced` (artist → artist)
  - `member_of` (artist ↔ school)
  - `studied_at` (artist ↔ institution)
  - `movement_member` (artist ↔ movement)

### Step 3 — Network analysis
- **Artist influence layer**: extract `G_inf` containing only artist→artist `influenced` edges.
  - Metrics: out-degree / in-degree, PageRank, betweenness centrality.
- **Movement / School / Institution “influence”** via *projection graphs*:
  - Build bipartite graph (artist ↔ movement/school/institution)
  - Project to movement–movement (or school–school, institution–institution)
  - Edge weight = number of **shared artists** (how strongly two entities overlap)
  - Rank entities by degree / weighted degree.
- **Nationality concentration**
  - Create nationality↔artist bipartite links
  - Report concentration metrics and coverage of top-n groups.
- **Community detection**
  - Run greedy modularity to find large communities and inspect examples.
- **Visualization**
  - Static ego-graphs around a chosen artist (matplotlib)
  - Interactive graph view (plotly)

---

## Results snapshot (from a sample run)

These are the key numbers printed by the notebook in the provided run:

### Graph size
- After loading base entities: **3,289 nodes**
- After processing relationships: **3,516 nodes**, **6,649 edges**
- After adding movements: **3,693 nodes**, **15,175 edges**
- Node types in final view:
  - **3,155 artists**
  - **273 schools**
  - **177 movements**
  - **88 institutions**

### Relationship coverage
- Relationship rows with at least one potential edge: **2,973 / 2,996 (99.2%)**
- Edge counts by relation type (before movement-membership expansion):
  - `member_of`: **2,652**
  - `studied_at`: **1,670**
  - `influenced`: **1,396**
  - `friend`: **931**
- Influence subgraph (`artist → artist`): **800 nodes**, **1,396 edges**

### Movements
- Total movement mentions in metadata: **4,263**
- Artists with ≥1 movement: **2,956**
- Normalized unique movements: **177**
- Movement membership edges added (directed): **8,526**

### Nationalities
- Top 10 nationalities cover: **64.82%** of artists
- Top 20 nationalities cover: **78.48%** of artists
- Nationality concentration (HHI): **0.0664** *(0 = very spread, 1 = monopoly)*

### Communities (greedy modularity)
Top community sizes in the run: **141, 126, 104, 81, 78, 65, 46, 43, 41, 39**

---


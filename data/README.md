# 📁 Data

Curated outputs of the pipeline. All data derives from **public sources** — [DBLP](https://dblp.org/) (open metadata) and [OpenAlex](https://openalex.org/) (**CC0 / public domain**).

> ℹ️ Paper **abstracts are intentionally not included**. OpenAlex distributes abstracts only as an *inverted index* to respect publishers' copyright, so the reconstructed full text is **not** redistributed here.

---

## `03_5_gold_nlp_validated.csv` — the curated corpus (438 dataset-papers)

The **Gold layer**: papers validated as *dataset-papers* by the zero-shot NLP classifier.

| Column | Description |
|---|---|
| `Conferenza` | Venue — KDD / CIKM / ICWSM |
| `Anno` | Publication year |
| `Categoria` | DBLP editorial track |
| `Titolo` | Paper title |
| `DOI` | DOI |
| `Citazioni` | Global citation count (OpenAlex) |
| `Open_Access` | Open Access flag |
| `Score_Semantico` | Heuristic repository/semantic score |
| `Repository_Links` | Repository URLs found in the abstract |
| `NLP_Dataset_Probability` | Zero-shot probability of the *"dataset"* label |
| `NLP_Predicted_Class` | Predicted class |

## Citation network — `05_citation_network_nodes.csv` · `05_citation_network_edges.csv`

Directed graph *"who cites whom"* — **6,581 nodes** (411 datasets + 6,170 citing papers), **6,613 edges**.

- **nodes**: `Id` (OpenAlex ID), `Label` (title), `Year`, `Node_Type` (*Original Dataset* / *Citing Paper*), `Conferenza`, `Global_Citations`, `In_Degree`, `In_Degree_Centrality`, `PageRank`
- **edges**: `Source`, `Target` (citing → dataset), `Type`

## Co-citation network — `06_cocitation_nodes.csv` · `06_cocitation_edges.csv` · `06_cocitation_network.gexf`

Datasets linked when a paper cites them together — **118 nodes**, **159 edges**, **25 thematic communities** (Louvain, modularity **0.738**).

- **nodes**: `Id`, `Label`, `Community`, `Betweenness`, `Degree`
- **edges**: `Source`, `Target`, `Weight`, `Type`
- **`.gexf`**: ready to open directly in **[Gephi](https://gephi.org/)**.

---

### Sources
- **[DBLP](https://dblp.org/)** — conference metadata (titles, DOIs, venues).
- **[OpenAlex](https://openalex.org/)** — citations, Open Access status, work IDs (CC0).

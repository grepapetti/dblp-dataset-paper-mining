# 📊 DBLP Dataset-Paper Mining

### An ETL pipeline for mining *dataset-papers* from DBLP: web scraping, NLP validation, and citation-network analysis

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/NLP-BART--large--MNLI-FF6F00?logo=huggingface&logoColor=white" alt="NLP">
  <img src="https://img.shields.io/badge/Network-NetworkX%20%7C%20Gephi-2C5985" alt="Network">
  <img src="https://img.shields.io/badge/Data-OpenAlex%20%7C%20DBLP-009E73" alt="Data">
</p>

> **Web & Data Science project — Politecnico di Milano**
> Supervisor: **Prof. Francesco Pierri**

---

## 🎯 Goal

Who releases **datasets** in research, and how are they reused? This project builds an **end-to-end ETL pipeline** that starts from the public **DBLP** indexes, automatically identifies the *papers that introduce a dataset* across three venues — **KDD, CIKM, and ICWSM (2015–2025)** — enriches them with metadata and citations from **OpenAlex**, validates them with a **zero-shot NLP** model, and finally analyzes the resulting **citation and co-citation networks** to surface hubs, thematic communities, and bridge datasets.

The outcome is a curated corpus of **438 high-precision dataset-papers** (out of ~11,800 raw records) and a full network analysis of the ecosystem.

---

## 🏗️ Architecture — a three-layer (Medallion) pipeline

```
   DBLP (HTML)          OpenAlex API           BART zero-shot         NetworkX + Gephi
       │                     │                       │                      │
   ┌───▼────┐          ┌─────▼─────┐           ┌──────▼──────┐        ┌──────▼──────┐
   │ BRONZE │ ───────▶ │  SILVER   │ ────────▶ │    GOLD     │ ─────▶ │   NETWORK   │
   │scraping│          │enrichment │           │ validation  │        │  analysis   │
   └────────┘          └───────────┘           └─────────────┘        └─────────────┘
  11,806 raw          +citations, OA,          438 validated          citation +
   records             abstracts                dataset-papers (3.71%) co-citation nets
```

### 🥉 Bronze — Extraction from DBLP
Structured web scraping of the conference indexes with `requests` + `BeautifulSoup`, following strict **netiquette**:
- **`robots.txt`** checked with `urllib.robotparser` before any access;
- **descriptive User-Agent** with a contact address (no anonymous scraping);
- **robust rate-limiting**: progressive linear backoff (10s → 20s → … → 50s) on HTTP `503`, `crawl-delay` respected, single keep-alive session.

Title, year, canonical DOI, and editorial track are extracted from each entry.

### 🥈 Silver — Enrichment via OpenAlex
Each paper is enriched by querying **OpenAlex** (in the *polite pool* via `mailto`, up to 10 req/s):
- **batches of 50 DOIs** per call (~11,000 papers in ~220 requests);
- **exact title fallback** (`filter=title.search` + similarity threshold ≥ 0.85) for papers without a DOI;
- **abstract reconstructed** from OpenAlex's *inverted index*;
- retry with backoff on `429/503`, incremental checkpoints every 1,000 records.

→ *11,517 abstracts recovered, 5,563 Open Access.*

### 🥇 Gold — NLP validation
Brittle keyword rules are replaced by **semantic understanding** with a **zero-shot** classifier (`facebook/bart-large-mnli`), which scores each paper against three candidate labels (*releases a dataset* / *proposes a method* / *is a survey*). Validation rules:
- **high confidence**: probability ≥ 0.65 → dataset-paper;
- **moderate confidence**: ≥ 0.50 **only if** the paper is in a *Dataset* or *Resource/Tools Track*.

→ **438 validated dataset-papers** — CIKM 185 · ICWSM 161 · KDD 92.

---

## 🕸️ Network Analysis

**Citation network** — built from OpenAlex (for each dataset, all the papers that cite it):
- **6,581 nodes** (411 datasets + 6,170 citing papers) · **6,613 edges**;
- metrics computed in **NetworkX** (in-degree, in-degree centrality, PageRank, betweenness, community detection via **Louvain**);
- visualized in **Gephi** (ForceAtlas2 spatialization, color = node type / community).

**Co-citation network** — two datasets are linked if a paper cites them together:
- **118 co-cited datasets** · **159 edges** · **25 thematic communities** (Louvain, modularity **0.738**);
- extraction of **bridge datasets** (high betweenness) and a **meta-network of communities**.

---

## 📈 Key findings

| Analysis | Insight |
|---|---|
| **Foundational hubs** | A few hubs concentrate most of the internal citations (power-law distribution): an anomaly-detection framework, CrisisMMD, CREDBANK. |
| **Thematic communities** | A dense *social-media / news / disinformation* core is co-cited tightly; most communities stay isolated → a fragmented ecosystem. |
| **Bridge datasets** | Brokers (e.g. VoterFraud2020, TwiBot-20, KuaiSAR) connect the *social/political*, *recommendation/search*, and *fake-news* poles. |
| **Disciplinary silos** | **59%** of co-citations are between datasets from the **same venue** — communities remain fairly closed. |
| **Data Citation Advantage** | Once maturity and editorial bias are controlled for, citation medians of papers *with* vs *without* a dataset **align** (12 vs 12, p = 0.73): releasing a dataset is now a *standard*, not a citation lever. |
| **Open Access** | OA adoption of datasets grows sharply: **40% (2015) → 85% (2025)**. |

---

## 🛠️ Tech stack

| Area | Tools |
|---|---|
| Scraping | `requests`, `BeautifulSoup`, `urllib.robotparser` |
| Data & API | `pandas`, `tqdm`, **OpenAlex API** |
| NLP | `transformers` — `facebook/bart-large-mnli` (zero-shot), `torch` |
| Network | `NetworkX`, **Gephi** (ForceAtlas2), Louvain |
| Visualization | `plotly` (Okabe-Ito, color-blind-safe palette) |

---

## 📂 Repository structure

```
dblp-dataset-paper-mining/
├── README.md
├── data/                                    # curated datasets (see data/README.md)
│   ├── 03_5_gold_nlp_validated.csv          #   → 438 validated dataset-papers
│   ├── 05_citation_network_nodes.csv        #   → citation network (6,581 nodes)
│   ├── 05_citation_network_edges.csv        #   → citation network (6,613 edges)
│   ├── 06_cocitation_nodes.csv              #   → co-citation network (118 nodes)
│   ├── 06_cocitation_edges.csv              #   → co-citation network (159 edges)
│   └── 06_cocitation_network.gexf           #   → co-citation graph, open in Gephi
└── notebook/
    └── ScrapingDBLP_pipeline.ipynb          # full pipeline (Bronze → Silver → Gold → Network)
```

The notebook is organized into modules, one per pipeline stage, with outputs and figures already executed.

## 📁 Data

The **[`data/`](data/)** folder ships the curated outputs so the analysis is **reproducible without re-scraping**: the 438 validated dataset-papers, and the citation & co-citation networks (also as a Gephi `.gexf`). See **[`data/README.md`](data/README.md)** for the column schema.

Everything comes from public sources (DBLP, OpenAlex/CC0). Paper **abstracts are not redistributed** — OpenAlex serves them only as an inverted index to respect copyright.

---

## ▶️ How to run

1. Open `notebook/ScrapingDBLP_pipeline.ipynb` (**Google Colab** or Jupyter with a GPU recommended for the NLP step).
2. Set your email for the OpenAlex *polite pool*:
   ```python
   USER_EMAIL = "your-email@example.com"   # ← put your email here
   ```
3. Run the modules in order (Bronze → Silver → Gold → Network).

> ⚠️ The OpenAlex steps are subject to the API's rate-limit / daily budget.

---

## 📚 Data sources

- **[DBLP](https://dblp.org/)** — conference indexes (primary source of papers).
- **[OpenAlex](https://openalex.org/)** — citations, Open Access status, abstracts (open scholarly index).

---

## 👤 Author

Thesis project in **Web & Data Science**, Politecnico di Milano.
Supervisor: **Prof. Francesco Pierri**.

<sub>Data comes from public sources (DBLP, OpenAlex) and is used for academic research purposes.</sub>

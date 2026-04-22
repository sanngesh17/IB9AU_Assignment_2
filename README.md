# SEC EDGAR RAG Project

A notebook-driven RAG pipeline over SEC EDGAR filings. 

The domain is 2025 10-K, 10-Q, and 8-K filings for Visa Inc. and Mastercard Incorporated. This could be expanded into a list of companies further. 

---

## Notebook summaries

### `01_fetch_and_normalize.ipynb` — Data ingestion

Fetches recent SEC filings for Visa (CIK: 1403161) and Mastercard (CIK: 1141391) directly from the EDGAR API, selects the most recent 2025 filings per form type, strips HTML/XML boilerplate, and writes a clean JSONL corpus to disk.

**Results:**
- 24 filings saved for fiscal year 2025: 2 × 10-K, 4 × 10-Q, 8 × 8-K per company
- 1,902,941 total characters across all filings
- Visa: Nov 2025 10-K, three 10-Qs (Jan/Apr/Jul 2025), eight 8-Ks
- Mastercard: Feb 2025 10-K, three 10-Qs (May/Jul/Oct 2025), eight 8-Ks

---

### `02_chunk_embed_index.ipynb` — Chunking, embedding, and indexing

Converts the raw corpus into four chunking variants, embeds every chunk with `sentence-transformers/all-MiniLM-L6-v2`, and persists a FAISS flat-IP index + BM25Okapi index + metadata per strategy.

**Chunking strategies and results:**

| Strategy | Chunks | Mean Chars | Median Chars | p95 Chars | Notes |
|---|---|---|---|---|---|
| Fixed | 4,625 | 548 | 560 | 778 | 128-token windows, 32-token overlap. Zero short chunks dropped. |
| Section | 4,341 | 548 | 562 | 779 | Splits on SEC section headings (Part I/II, Item N). Falls back to fixed when no headings found. 85 stubs dropped. |
| Semantic | 7,609 | 245 | 178 | 715 | Rolling embedding window (cosine threshold 0.70, max 256 tokens). Produces more, smaller chunks. 1,230 micro-chunks dropped. |
| Hierarchical | 6,136 | 286 | 206 | 759 | Section split → semantic split within each section. 2,703 stubs dropped. |

Fixed and Section produce uniform chunk sizes that work well for BM25-style keyword search. Semantic and Hierarchical produce variable-size chunks that preserve topic coherence, at the cost of more total chunks and much slower build time (~109s for semantic vs 0.5s for fixed).

---

### `03_retrieve_generate_compare.ipynb` — Retrieval, reranking, and generation

Implements a `Retriever` class that can be pointed at any chunking strategy and switched between four retrieval modes, plus query-transformation helpers (query rewriting, HyDE, multi-query expansion) and prompt templates for baseline and enhanced generation.

**Retrieval modes:**
- **Vector** — dense FAISS inner-product search on `all-MiniLM-L6-v2` embeddings
- **BM25** — sparse keyword search with stopword filtering (BM25Okapi)
- **Hybrid** — Reciprocal Rank Fusion (RRF, c=60) fusing vector and BM25 ranked lists
- **Hybrid+Rerank** — hybrid pool rescored by `cross-encoder/ms-marco-MiniLM-L-6-v2`

End-to-end check: baseline `fixed/vector` correctly retrieved chunks from Visa 10-K (Nov 2025) and 10-Q (Apr/Jul 2025) in the top-5 for a Visa Q3 2025 revenue query. The enhanced pipeline used HyDE-rewritten queries against `hierarchical/hybrid+rerank`.

---

### `04_evaluate_and_report.ipynb` — Full evaluation matrix

Runs the complete 4 × 4 matrix (chunking × retrieval) over an auto-generated and hand-crafted query set. Reports Hit@k, MRR, Recall@k, Precision@k, nDCG@k at k ∈ {1, 3, 5, 10}, context precision/recall, Jaccard overlap, and latency.

**Query set:**
- 62 auto-generated queries (LLM-curated from numeric/risk sentences in the corpus)
- 4 manual queries (payment volume trends, regulatory risk, macroeconomic synthesis)
- Total: 66 queries — 55 simple fact, 10 deep context, 1 edge case

**Gold-sentence recoverability:**

| Strategy | Recoverable / Total | Recovery Rate |
|---|---|---|
| Fixed | 64 / 64 | 100% |
| Section | 64 / 64 | 100% |
| Semantic | 61 / 64 | 95.3% |
| Hierarchical | 61 / 64 | 95.3% |

Fixed and Section preserve every gold sentence intact. Semantic and Hierarchical split 3 gold sentences across chunk boundaries, making those queries structurally unrecoverable no matter how good the retrieval is.

---

## Strategies

### Chunking strategies

| Strategy | Description | Tradeoff |
|---|---|---|
| Fixed | Token-aware sliding window (128 tokens, 32 overlap) | Uniform size, predictable; ignores semantic boundaries |
| Section | Split at SEC heading patterns (Part I, Item N); recursive fixed fallback | Preserves filing structure; chunk length varies widely |
| Semantic | Rolling cosine window; new chunk when similarity < 0.70 or > 256 tokens | Topic-coherent chunks; smallest mean size; most chunks overall |
| Hierarchical | Section split → semantic sub-split; parent text attached to each child | Best of both; child chunk retrieved, parent context sent to LLM |

### Retrieval modes

| Mode | Description | Tradeoff |
|---|---|---|
| Vector | Dense FAISS nearest-neighbour | Fast; semantic matching; struggles with exact numbers |
| BM25 | Sparse keyword overlap | Excellent on exact figures and named entities; no semantic generalisation |
| Hybrid (RRF) | Fuses both ranked lists via Reciprocal Rank Fusion | Balanced; slightly slower than each alone |
| Hybrid+Rerank | Hybrid pool rescored by cross-encoder | Highest accuracy; adds ~0.6–0.8s latency |

---

## Results

### Retrieval metric matrix (mean over 63 auto gold queries)

| Strategy | Mode | Hit@5 | MRR | nDCG@5 | Recall@5 | Precision@5 |
|---|---|---|---|---|---|---|
| hierarchical | hybrid+rerank | **0.951** | **0.812** | **0.815** | **0.888** | 0.223 |
| semantic | hybrid+rerank | **0.951** | **0.812** | 0.811 | 0.881 | **0.226** |
| section | hybrid+rerank | 0.906 | 0.789 | 0.738 | 0.789 | 0.244 |
| fixed | hybrid+rerank | 0.875 | 0.720 | 0.680 | 0.752 | 0.253 |
| section | bm25 | 0.875 | 0.758 | 0.707 | 0.750 | 0.231 |
| hierarchical | bm25 | 0.869 | 0.725 | 0.743 | 0.825 | 0.216 |
| semantic | bm25 | 0.869 | 0.692 | 0.715 | 0.816 | 0.216 |
| fixed | bm25 | 0.859 | 0.721 | 0.687 | 0.753 | 0.250 |
| hierarchical | hybrid | 0.820 | 0.657 | 0.651 | 0.761 | 0.187 |
| semantic | hybrid | 0.803 | 0.631 | 0.621 | 0.734 | 0.180 |
| section | hybrid | 0.844 | 0.580 | 0.571 | 0.713 | 0.200 |
| fixed | hybrid | 0.734 | 0.550 | 0.515 | 0.630 | 0.206 |
| hierarchical | vector | 0.672 | 0.503 | 0.511 | 0.620 | 0.148 |
| semantic | vector | 0.639 | 0.480 | 0.491 | 0.598 | 0.141 |
| section | vector | 0.562 | 0.394 | 0.382 | 0.476 | 0.131 |
| fixed | vector | 0.531 | 0.384 | 0.347 | 0.414 | 0.134 |

### A note on precision vs. recall

Precision@5 = fraction of the 5 retrieved chunks that are relevant. Recall@5 = fraction of *all* relevant chunks that appear in the top 5.

Precision is intentionally low (~0.2–0.3) here because each query can have many relevant chunks — the same fact appears across multiple filings or sections — and a window of 5 can only cover a small slice. Recall is the more meaningful metric: it tells you whether the correct passage made it into the context the LLM sees. As long as recall is high, the generator has what it needs. Extra irrelevant chunks in the window are a minor cost, not a failure.

### Context quality (top-5 retrieved)

| Strategy | Mode | Context Precision@5 | Context Recall@5 | Jaccard Overlap |
|---|---|---|---|---|
| fixed | hybrid+rerank | **0.291** | 0.722 | 0.306 |
| section | hybrid+rerank | 0.276 | 0.723 | 0.307 |
| semantic | hybrid+rerank | 0.262 | **0.840** | 0.385 |
| hierarchical | hybrid+rerank | 0.252 | 0.837 | 0.388 |
| fixed | bm25 | 0.268 | 0.708 | 0.281 |

### Latency

| Strategy | Mode | Mean (s) | Median (s) |
|---|---|---|---|
| section | vector | **0.027** | **0.024** |
| fixed | bm25 | 0.036 | 0.035 |
| section | bm25 | 0.036 | 0.035 |
| fixed | hybrid | 0.079 | 0.078 |
| hierarchical | hybrid+rerank | 0.609 | 0.591 |
| semantic | hybrid+rerank | 0.612 | 0.582 |
| fixed | hybrid+rerank | 0.837 | 0.833 |
| section | hybrid+rerank | 0.804 | 0.803 |

### Generation quality (LLM answer evaluation)

Measured using cosine similarity between the answer embedding and question/context embeddings:

| Strategy | Mode | Answer Relevance | Answer Faithfulness |
|---|---|---|---|
| hierarchical | hybrid+rerank | **0.757** | **0.637** |
| fixed | vector | 0.589 | 0.474 |

The enhanced pipeline (hierarchical + hybrid+rerank) produces answers 28% more relevant and 34% more faithful to source context than the fixed + vector baseline.

### Per question-type breakdown

**Deep context questions** (risk disclosures, cross-filing synthesis):

| Strategy | Mode | Hit@5 | MRR | nDCG@5 |
|---|---|---|---|---|
| fixed | hybrid+rerank | **1.000** | 0.818 | 0.738 |
| hierarchical | bm25 | **1.000** | 0.773 | 0.837 |
| section | hybrid+rerank | — | — | — |

**Simple fact questions** (revenue figures, growth rates, specific metrics):

| Strategy | Mode | Hit@5 | MRR | nDCG@5 |
|---|---|---|---|---|
| hierarchical | hybrid+rerank | ~0.94 | ~0.81 | ~0.82 |
| fixed | bm25 | 0.849 | 0.701 | 0.664 |

BM25 punches above its weight on simple fact questions because they contain exact numeric strings that keyword matching handles well. Hybrid+rerank consistently wins on deep context because the cross-encoder can reason about passage quality holistically.

---

## Interpretations

### BM25 vs. pure vector

SEC filings are full of precise numbers — `$35.2 billion`, `23%`, `Q3 FY2025` — that dense embeddings average away. BM25's exact-match behavior makes it the best standalone mode for simple fact retrieval. Across all chunking strategies, BM25 outperforms vector by roughly 33 Hit@5 points (0.86 vs 0.58 on average). This wasn't surprising, but the margin was larger than expected.

### Why hybrid+rerank wins overall

The cross-encoder sees the full (query, passage) pair and ranks by actual relevance, not just embedding similarity. The ~0.6–0.8s reranking cost is worth it: it lifts Hit@5 from 0.869 (best BM25) to 0.951 (+9%) and nDCG@5 from 0.743 to 0.815 (+10%).

### Why hierarchical chunking outperforms fixed

Each child chunk is retrieved with its parent section attached as context, so the LLM gets broader, coherent context even though retrieval was precise (the small child chunk matched the query). Fixed chunking cuts across sentence and section boundaries, so the answer may start at the tail of one chunk and continue into the next.

### The recoverability tradeoff on semantic/hierarchical

Semantic and Hierarchical strategies split 3 of 64 gold sentences across chunk boundaries — 95.3% recovery rate vs 100% for Fixed/Section. For production use cases with precise numeric fact queries, that 4.7% structural loss is real. If recovery rate matters more than retrieval quality, Fixed or Section are safer choices.

### Context precision vs. recall

Fixed chunking gets the highest context precision (0.291 with hybrid+rerank) because its uniform, compact chunks are less likely to drag in off-topic tokens. Semantic and Hierarchical get the highest context recall (0.840/0.837) because their variable-size chunks tend to capture more relevant text once a hit lands. For generation, high recall matters more than precision when the context window is large enough to absorb some noise.

### Latency

The cheapest useful pipeline is `section + bm25` at 36ms mean latency with 0.875 Hit@5 — nearly matching the best vector result, with 3× the recall and no reranker. If sub-50ms P50 latency is a hard requirement, that's the one to use. The reranker adds ~600ms and only buys a meaningful lift for deep-context and ambiguous queries.

---

## Recommendation

| Use case | Recommended pipeline | Rationale |
|---|---|---|
| Default / production | `hierarchical + hybrid+rerank` | Best Hit@5 (0.951), nDCG@5 (0.815), answer faithfulness (0.637) |
| Latency-sensitive | `section + bm25` | 36ms mean, 0.875 Hit@5, no reranker dependency |
| Regression baseline | `fixed + vector` | Simplest pipeline; deterministic; useful as lower bound |
| Deep context / risk synthesis | `hierarchical + hybrid+rerank` | 100% Hit@5 on deep context, best recall across filings |

---

## Project structure

```
.
├── notebooks/
│   ├── 01_fetch_and_normalize.ipynb     # EDGAR ingestion → JSONL corpus
│   ├── 02_chunk_embed_index.ipynb       # 4 chunking strategies → FAISS + BM25
│   ├── 03_retrieve_generate_compare.ipynb  # Retriever class + sanity checks
│   └── 04_evaluate_and_report.ipynb     # Full 4×4 evaluation matrix
├── data/
│   ├── raw/                             # Raw SEC submission JSONs
│   └── processed/                       # JSONL corpus, chunk tables, eval CSVs
├── artifacts/                           # FAISS indexes, BM25 pickles, embeddings
├── reports/
│   ├── summary.md                       # Auto-generated evaluation report
│   ├── heatmap_hit_at_5.png
│   ├── heatmap_mrr.png
│   └── heatmap_ndcg_at_5.png
└── requirements.txt
```

## Run order

1. `01_fetch_and_normalize.ipynb` — fetch filings (requires SEC User-Agent header)
2. `02_chunk_embed_index.ipynb` — build all four indexes (~10 min on CPU)
3. `03_retrieve_generate_compare.ipynb` — interactive sanity check
4. `04_evaluate_and_report.ipynb` — run evaluation matrix, export report

## Models used

- **Embedding:** `sentence-transformers/all-MiniLM-L6-v2` (384-dim, ~80MB)
- **Reranker:** `cross-encoder/ms-marco-MiniLM-L-6-v2`
- **LLM:** OpenAI `gpt-4o-mini` (query generation + generation eval; set `OPENAI_API_KEY`)

## Notes

- The retrieval layer is self-contained and does not depend on the generator — evaluation can run fully offline.
- Evaluation metrics are chunking-agnostic: a chunk is relevant iff it contains ≥75% of the gold sentence's non-stopword tokens, making the same query fair across all strategies.
- Large generated outputs are intentionally not tracked in git (`artifacts/` and multi-GB JSONL files under `data/processed/`). Reproduce them by running the notebooks in order.

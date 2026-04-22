# SEC EDGAR RAG — Evaluation Report

Auto-generated gold queries: 63. Retrieval grid: 4 chunking strategies x 4 retrieval modes.

## Headline findings

- Best Hit@5: **hierarchical / hybrid+rerank** at 0.951
- Best nDCG@5: **hierarchical / hybrid+rerank** at 0.815
- Fastest mean latency: **section / vector** at 0.027s

## Chunk-level statistics

    strategy  num_chunks  mean_tokens  median_tokens  p95_tokens  max_tokens  chunks_over_512_tokens  pct_over_512
       fixed        4625        127.8          128.0       128.0         128                       0           0.0
     section        4341        127.8          128.0       128.0         128                       0           0.0
    semantic        7609         57.2           33.0       256.0         256                       0           0.0
hierarchical        6136         67.0           39.0       256.0         256                       0           0.0

## Gold-sentence recoverability per chunking strategy

If the gold sentence cannot be contained by any single chunk, a query is unrecoverable for that strategy. This is itself a chunking-quality signal.

    strategy  auto_recoverable  auto_total  auto_recovery_rate
       fixed                64          64               1.000
     section                64          64               1.000
    semantic                61          64               0.953
hierarchical                61          64               0.953

## Retrieval metric matrix (mean over auto gold queries)

    strategy          mode   mrr  hit@1  hit@3  hit@5  hit@10  precision@1  precision@3  precision@5  precision@10  recall@1  recall@3  recall@5  recall@10  ndcg@1  ndcg@3  ndcg@5  ndcg@10
       fixed          bm25 0.721  0.625  0.828  0.859   0.875        0.625        0.375        0.250         0.139     0.415     0.699     0.753      0.794   0.625   0.668   0.687    0.697
       fixed        hybrid 0.550  0.391  0.641  0.734   0.875        0.391        0.266        0.206         0.133     0.246     0.485     0.630      0.786   0.391   0.456   0.515    0.569
       fixed hybrid+rerank 0.720  0.594  0.828  0.875   0.891        0.594        0.370        0.253         0.142     0.375     0.690     0.752      0.824   0.594   0.656   0.680    0.702
       fixed        vector 0.384  0.297  0.422  0.531   0.625        0.297        0.161        0.134         0.083     0.200     0.315     0.414      0.488   0.297   0.301   0.347    0.373
hierarchical          bm25 0.725  0.623  0.803  0.869   0.885        0.623        0.301        0.216         0.123     0.570     0.740     0.825      0.846   0.623   0.710   0.743    0.748
hierarchical        hybrid 0.657  0.525  0.754  0.820   0.918        0.525        0.262        0.187         0.113     0.472     0.666     0.761      0.862   0.525   0.616   0.651    0.687
hierarchical hybrid+rerank 0.812  0.721  0.902  0.951   0.951        0.721        0.339        0.223         0.125     0.649     0.841     0.888      0.908   0.721   0.800   0.815    0.821
hierarchical        vector 0.503  0.377  0.590  0.672   0.738        0.377        0.208        0.148         0.087     0.341     0.535     0.620      0.710   0.377   0.477   0.511    0.541
     section          bm25 0.758  0.672  0.828  0.875   0.906        0.672        0.349        0.231         0.136     0.494     0.696     0.750      0.822   0.672   0.689   0.707    0.729
     section        hybrid 0.580  0.438  0.641  0.844   0.891        0.438        0.250        0.200         0.122     0.338     0.542     0.713      0.784   0.438   0.497   0.571    0.598
     section hybrid+rerank 0.789  0.703  0.875  0.906   0.922        0.703        0.359        0.244         0.138     0.508     0.734     0.789      0.847   0.703   0.717   0.738    0.755
     section        vector 0.394  0.281  0.500  0.562   0.625        0.281        0.193        0.131         0.080     0.224     0.430     0.476      0.545   0.281   0.360   0.382    0.408
    semantic          bm25 0.692  0.590  0.787  0.869   0.869        0.590        0.290        0.216         0.118     0.529     0.722     0.816      0.826   0.590   0.675   0.715    0.714
    semantic        hybrid 0.631  0.492  0.738  0.803   0.918        0.492        0.262        0.180         0.115     0.442     0.653     0.734      0.862   0.492   0.594   0.621    0.668
    semantic hybrid+rerank 0.812  0.721  0.902  0.951   0.951        0.721        0.344        0.226         0.126     0.641     0.841     0.881      0.908   0.721   0.798   0.811    0.819
    semantic        vector 0.480  0.361  0.574  0.639   0.689        0.361        0.202        0.141         0.082     0.325     0.519     0.598      0.653   0.361   0.461   0.491    0.509

## Context quality (top-5)

    strategy          mode  context_precision@5  context_recall@5  top5_jaccard_overlap
       fixed          bm25                0.268             0.708                 0.281
       fixed        hybrid                0.250             0.631                 0.274
       fixed hybrid+rerank                0.291             0.722                 0.306
       fixed        vector                0.174             0.390                 0.267
hierarchical          bm25                0.215             0.774                 0.374
hierarchical        hybrid                0.215             0.717                 0.358
hierarchical hybrid+rerank                0.252             0.837                 0.388
hierarchical        vector                0.185             0.597                 0.399
     section          bm25                0.259             0.706                 0.279
     section        hybrid                0.238             0.671                 0.258
     section hybrid+rerank                0.276             0.723                 0.307
     section        vector                0.182             0.448                 0.278
    semantic          bm25                0.212             0.766                 0.374
    semantic        hybrid                0.212             0.707                 0.342
    semantic hybrid+rerank                0.262             0.840                 0.385
    semantic        vector                0.175             0.562                 0.381

## Latency (seconds)

    strategy          mode   mean  median
       fixed          bm25 0.0359  0.0351
       fixed        hybrid 0.0793  0.0784
       fixed hybrid+rerank 0.8369  0.8326
       fixed        vector 0.0579  0.0306
hierarchical          bm25 0.0457  0.0426
hierarchical        hybrid 0.0843  0.0839
hierarchical hybrid+rerank 0.6085  0.5907
hierarchical        vector 0.0538  0.0223
     section          bm25 0.0362  0.0346
     section        hybrid 0.0827  0.0798
     section hybrid+rerank 0.8044  0.8031
     section        vector 0.0270  0.0244
    semantic          bm25 0.0546  0.0500
    semantic        hybrid 0.0987  0.0968
    semantic hybrid+rerank 0.6118  0.5820
    semantic        vector 0.0608  0.0242

## Per question-type breakdown (hit@5, MRR, nDCG@5)

    strategy          mode question_type  hit@5   mrr  ndcg@5
       fixed          bm25  deep context  0.909 0.818   0.794
       fixed          bm25   simple fact  0.849 0.701   0.664
       fixed        hybrid  deep context  0.909 0.657   0.673
       fixed        hybrid   simple fact  0.698 0.527   0.482
       fixed hybrid+rerank  deep context  1.000 0.818   0.738
       fixed hybrid+rerank   simple fact  0.849 0.700   0.667
       fixed        vector  deep context  0.636 0.483   0.423
       fixed        vector   simple fact  0.509 0.363   0.331
hierarchical          bm25  deep context  1.000 0.773   0.837
hierarchical          bm25   simple fact  0.840 0.714   0.722
hierarchical        hybrid  deep context  0.818 0.624   0.654
hierarchical        hybrid   simple fact  0.820 0.664   0.650
hierarchical hybrid+rerank  deep context  1.000 0.848   0.884
hierarchical hybrid+rerank   simple fact  0.940 0.804   0.799
hierarchical        vector  deep context  0.545 0.452   0.466
hierarchical        vector   simple fact  0.700 0.514   0.521
     section          bm25  deep context  0.909 0.758   0.707
     section          bm25   simple fact  0.868 0.759   0.707
     section        hybrid  deep context  0.909 0.636   0.612
     section        hybrid   simple fact  0.830 0.568   0.562
     section hybrid+rerank  deep context  1.000 0.818   0.730
     section hybrid+rerank   simple fact  0.887 0.783   0.740
     section        vector  deep context  0.636 0.530   0.465
     section        vector   simple fact  0.547 0.365   0.365
    semantic          bm25  deep context  1.000 0.742   0.819
    semantic          bm25   simple fact  0.840 0.681   0.692
    semantic        hybrid  deep context  0.727 0.589   0.593
    semantic        hybrid   simple fact  0.820 0.640   0.627
    semantic hybrid+rerank  deep context  1.000 0.788   0.839
    semantic hybrid+rerank   simple fact  0.940 0.817   0.804
    semantic        vector  deep context  0.545 0.439   0.466
    semantic        vector   simple fact  0.660 0.489   0.496

## How to read this

- **Hit@5 / MRR** tell you whether the right passage is somewhere near the top. Hybrid+rerank typically wins here.
- **nDCG@5** penalises putting relevant chunks below less-relevant ones — the most faithful single summary metric.
- **Recall@5** matters more for multi-hop, deep-context questions (e.g. cross-filing risk synthesis).
- **Context precision / top5 Jaccard overlap** predict generation cost: high overlap means you are paying for duplicate tokens.
- **Recoverability** exposes chunks that split across gold sentences — a structural loss no retrieval method can recover.
- **Latency** is dominated by the reranker. If the nDCG lift is <0.02 over plain hybrid, reranking is usually not worth the cost.

## Recommendation

**hierarchical + hybrid+rerank** as the default pipeline; keep **fixed + vector** as the baseline for regression tests.
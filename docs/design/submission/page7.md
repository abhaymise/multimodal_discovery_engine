# Production Architecture, Latency, Cost & Evaluation

# 1. End-to-end production architecture

```text
                              CATALOG
                                 │
                          CDC / Event Bus
                                 │
                                 ▼
                     ┌─────────────────────┐
                     │ INGESTION PIPELINE  │
                     │ Kafka + Workers     │
                     └──────────┬──────────┘
                                │
                 ┌──────────────┼──────────────┐
                 ↓              ↓              ↓
              Metadata        Images         Video
                 │              │              │
                 │         Offline Enrichment  │
                 │              │              │
                 │      ┌───────┴────────┐     │
                 │      │ OCR / TSR / VLM│     │
                 │      │ Attributes     │     │
                 │      │ Captions       │     │
                 │      └───────┬────────┘     │
                 │              │              │
                 └──────────────┼──────────────┘
                                ↓
                     Canonical Product Model
                                │
             ┌──────────────────┼──────────────────┐
             ↓                  ↓                  ↓
       Elasticsearch        Vector DB            Redis
       BM25 + filters      Multi-vector ANN      Hot features
             │                  │                  │
             └──────────────────┼──────────────────┘
                                │
                         ───── ONLINE ─────
                                │
                      Text/Image/Video Query
                                │
                                ▼
                     Query Understanding
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
           BM25/ES             ANN           Structured
              │                 │             Filters
              └─────────────────┼─────────────────┘
                                ↓
                         Candidate Fusion
                                ↓
                          Lightweight LTR
                                ↓
                            TOP-K RESULTS
```

---

# 2. Latency budget

The system should target approximately **95–105 ms P99**, leaving ~15–25 ms headroom under the 120 ms requirement.

| Component | Target P99 |
|---|---:|
| Gateway / network | 5–10 ms |
| Query understanding | 5–10 ms |
| Query embedding | 5–10 ms |
| Elasticsearch | 10–15 ms |
| ANN retrieval | 10–15 ms |
| Structured filtering | 3–5 ms |
| Candidate fusion | 3–5 ms |
| LTR | 10–15 ms |
| Serialization | 3–5 ms |
| **Target** | **~55–90 ms compute** |
| Headroom | **30–65 ms** |

The exact numbers require load testing, but the architecture deliberately keeps expensive model inference out of the synchronous path.

---

# 3. Online compute budget

The <$5K/month target drives several design decisions.

### Avoid

```text
Every query
    ↓
Large VLM
    ↓
Cross encoder over hundreds of products
```

This is operationally incompatible with the budget.

### Prefer

```text
Query
 ↓
Small encoder
 ↓
ANN
 ↓
BM25
 ↓
Structured filters
 ↓
Small LTR
```

Heavy models are used during:

- Initial catalog enrichment
- New product enrichment
- Periodic re-enrichment
- Teacher-label generation
- Distillation

---

# 4. Near-real-time freshness

A new product should not wait for a nightly batch.

```text
NEW PRODUCT
    │
    ↓
Catalog CDC
    │
    ↓
Kafka
    │
    ├──────────────→ Fast metadata indexing
    │
    ↓
Priority enrichment queue
    │
    ↓
Lightweight enrichment
    │
    ├── OCR
    ├── compact image encoder
    ├── attribute extraction
    └── semantic representation
    │
    ↓
Incremental index update
    │
    ├── Elasticsearch
    ├── Vector DB
    └── Redis
    │
    ↓
SEARCHABLE
```

### Two-tier enrichment

**Tier 1 — Fast path**

Make the product discoverable quickly using:

- existing metadata
- image embedding
- basic visual attributes
- OCR
- taxonomy

**Tier 2 — Deep enrichment**

Run asynchronously:

- expensive VLM
- detailed attribute extraction
- TSR
- semantic concepts
- richer descriptions

This gives us **fast freshness without sacrificing eventual quality**.

---

# 5. Search-quality evaluation without ground truth

The absence of human labels does not mean the system cannot be evaluated.

## Synthetic evaluation

Use the strong offline teacher to create pseudo-ground truth.

```text
Product
   │
   ↓
Teacher VLM
   │
   ↓
Generate plausible queries
   │
   ↓
Known product = positive
   │
   ↓
Search engine
   │
   ↓
Recall@K / NDCG@K
```

Examples:

```text
Product → "summer beach dress"
Product → "floral vacation maxi dress"
Product → "bohemian beach outfit"
```

Evaluate whether the source product appears in top-K.

---

## Human evaluation

Create a curated benchmark across:

### Semantic

- "summer beach party"

### Visual

- image-to-product

### Technical

- "10mm hex bolt"

### Attribute

- "ribbed silk"

### Long-tail

- rare terminology

### Cross-modal

- text → video
- image → product
- video → product

---

# 6. Metrics

### Retrieval

- Recall@10
- Recall@50
- Recall@100
- MRR
- NDCG@10

### Attribute accuracy

- Precision / Recall / F1
- exact-match accuracy
- numeric extraction accuracy

### Multimodal

- Text → Image Recall@K
- Image → Product Recall@K
- Video → Product Recall@K

### Production

- P50 / P95 / P99 latency
- QPS
- zero-result rate
- query reformulation rate
- cache hit rate
- index freshness lag

### Business

- CTR
- Add-to-cart rate
- Conversion rate
- Revenue/session

---

# 7. Online experimentation

Use progressive rollout:

```text
Baseline
   ↓
5% traffic
   ↓
10%
   ↓
25%
   ↓
50%
   ↓
100%
```

Compare:

```text
Dense-only
vs
BM25-only
vs
Hybrid
vs
Hybrid + LTR
vs
Hybrid + multimodal enrichment
```

This allows us to isolate where quality gains originate.

---

# FINAL ARCHITECTURAL DECISION
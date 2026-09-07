# Problem, Constraints & Architecture Thesis

## 1. Problem

We operate a marketplace with:

- **5M+ products**
- ~95% of products have weak/missing textual descriptions
- Almost every product has **images**
- Many products have **5–10 second videos**
- Existing structured metadata is limited to:
  - Brand
  - Category
  - Price

Users need to discover products through:

1. Natural-language text
2. Image upload
3. Short video clip
4. Subjective semantic concepts such as *"90s retro"* or *"summer beach party"*
5. Fine-grained technical queries such as *"10mm hex bolt"* or *"ribbed silk shirt"*

### Hard production constraints

| Requirement | Target |
|---|---:|
| Catalog | 5M+ products |
| Search P99 | **<120 ms** |
| Online GPU/compute | **<$5K/month** |
| New-item discoverability | Near real time |
| Query modalities | Text + Image + Video |
| Retrieval | Semantic + lexical + structured |

---

## 2. Core architectural thesis

### Do NOT run expensive multimodal models synchronously.

Instead:

> **Use expensive foundation models offline to convert raw pixels into rich semantic representations, then distill/compile those representations into compact retrieval features and lightweight online models.**

This converts:

```text
EXPENSIVE COMPUTATION
       ↓
   OFFLINE
       ↓
Semantic Product Representation
       ↓
CHEAP COMPUTATION
       ↓
ONLINE SEARCH
```

---

## 3. Architecture at a glance

```text
                         CATALOG
                            │
             ┌──────────────┼──────────────┐
             │              │              │
          Metadata        Images          Video
             │              │              │
             └──────────────┼──────────────┘
                            ↓
                  ┌──────────────────┐
                  │ OFFLINE ENRICHER │
                  │                  │
                  │ OCR / TSR        │
                  │ VLM              │
                  │ Captioning       │
                  │ Attributes       │
                  │ Embeddings       │
                  └────────┬─────────┘
                           ↓
                Canonical Product Model
                           ↓
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
      Lexical          Vector Index     Structured
      Index             / ANN             Index
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                    ONLINE SEARCH
                           ↓
                Query Understanding
                           ↓
           ┌───────────────┼───────────────┐
           ↓               ↓               ↓
       Lexical          Dense          Structured
       Retrieval       Retrieval        Filtering
           └───────────────┼───────────────┘
                           ↓
                     Fusion / LTR
                           ↓
                         TOP-K
```

### Design principles

**1. Offline-heavy, online-light**

**2. Multi-vector instead of one universal embedding**

**3. Hybrid retrieval instead of dense-only retrieval**

**4. Exact attributes use structured/lexical retrieval**

**5. Semantic concepts use multimodal embeddings**

**6. Foundation models act as teachers; compact models serve online**

**7. Incremental indexing provides near-real-time freshness**

---


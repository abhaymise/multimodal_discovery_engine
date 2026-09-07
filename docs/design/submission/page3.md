# Canonical Product Representation & Feature Store

## 1. Canonical Product Representation

Every product is transformed into a common retrieval-oriented representation.

```text
                     PRODUCT
                        │
 ┌──────────────────────┼──────────────────────┐
 │                      │                      │
 ↓                      ↓                      ↓
STRUCTURED            TEXTUAL                VISUAL
 │                      │                      │
 ├─ category            ├─ title              ├─ image embeddings
 ├─ brand               ├─ description        ├─ visual attributes
 ├─ price               ├─ OCR                └─ object embeddings
 ├─ taxonomy            ├─ captions
 └─ business attrs      └─ attributes
                        │
                        ↓
                  SEMANTIC LAYER
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
          style       use-case   occasion
          vibe        audience   compatibility
             │
             └──────────┬──────────┘
                        ↓
                RETRIEVAL FEATURES
```

---

## 2. What should be precomputed?

### Precompute aggressively

Anything independent of the user query should be generated offline.

| Feature | Offline? | Online? |
|---|---:|---:|
| Image embedding | ✓ | |
| Video embedding | ✓ | |
| Generated title | ✓ | |
| Description | ✓ | |
| OCR | ✓ | |
| TSR | ✓ | |
| Product attributes | ✓ | |
| Style/vibe labels | ✓ | |
| Material | ✓ | |
| Object labels | ✓ | |
| Product semantic embedding | ✓ | |
| BM25 index | ✓ | |
| Taxonomy embedding | ✓ | |
| Popularity | ✓ | |
| Query embedding | | ✓ |
| Query intent | | ✓ |
| Query-specific similarity | | ✓ |
| LTR features | | ✓ |

---

## 3. Storage architecture

Do not force every feature into one database.

```text
                  PRODUCT FEATURE PIPELINE
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       Structured        Text             Vectors
          │                │                │
          ↓                ↓                ↓
     Cassandra /       Elasticsearch     Milvus /
     DynamoDB          / OpenSearch       Qdrant
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                       Redis
                  Hot online features
```

### Recommended responsibilities

**Object Storage**

- Raw images
- Videos
- Extracted frames
- Enrichment artifacts

**Iceberg/Parquet**

- Offline training datasets
- Historical features
- Teacher outputs
- Model-training data

**Elasticsearch/OpenSearch**

- BM25
- OCR text
- Generated title
- Attributes
- Exact keyword search
- Structured filtering

**Milvus/Qdrant**

- Dense ANN
- Image embeddings
- Text embeddings
- Product semantic embeddings

**Redis**

- Hot product features
- Product metadata cache
- Ranking features
- Query-result cache where useful

**Kafka**

- Catalog CDC
- Product updates
- Enrichment events
- Index update events

---

## 4. Why multiple vectors?

A single product vector is insufficient.

A product may have:

```text
Image representation
Text representation
Attribute representation
Style representation
Technical representation
Video representation
```

These should not necessarily collapse into one vector.

### Retrieval representation

```text
PRODUCT
  │
  ├── Product semantic vector
  ├── Image vector
  ├── Video vector
  ├── Attribute vector
  ├── Style/vibe vector
  └── Text/BM25 representation
```

This enables **multilevel + multivector retrieval**.

---

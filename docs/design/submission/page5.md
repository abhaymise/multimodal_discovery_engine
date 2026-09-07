# Query Understanding + Hybrid Retrieval

## 1. Online query architecture

The online path must remain extremely lightweight.

```text
                 USER QUERY
                     │
                     ↓
             Query Understanding
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     Intent       Attributes    Modality
       │             │             │
       ↓             ↓             ↓
   Query DSL     Structured       Encoder
                  constraints
       │             │             │
       └─────────────┼─────────────┘
                     ↓
             PARALLEL RETRIEVAL
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    BM25            ANN        Structured
       │             │          filtering
       └─────────────┼─────────────┘
                     ↓
               Candidate Fusion
                     ↓
                Lightweight LTR
                     ↓
                    TOP-K
```

---

## 2. Query understanding

Natural language should first be mapped to the **canonical product taxonomy and attribute schema**.

Example:

> "black waterproof hiking shoes under $100"

becomes:

```json
{
  "category": "hiking_shoes",
  "attributes": {
    "color": "black",
    "waterproof": true
  },
  "price": {
    "max": 100
  },
  "semantic_intent": "hiking"
}
```

Then compile into retrieval operations:

```text
category = hiking_shoes
AND color = black
AND waterproof = true
AND price <= 100
+
semantic("hiking")
```

---

## 3. Hybrid retrieval

The retrieval function is:

\[
Candidates =
Dense(q)
\cup
Lexical(q)
\cup
Structured(q)
\]

followed by ranking.

It is **not**:

\[
Dense(q) \cap Lexical(q)
\]

because requiring both would eliminate products that are semantically relevant but lack exact lexical matches.

---

## 4. Lexical retrieval

Use Elasticsearch/OpenSearch BM25 over:

```text
Generated title
Generated description
OCR
Product attributes
Brand
Category
Technical specification text
TSR-derived table text
Synonyms
Aliases
```

BM25 is particularly important for:

- IDs
- part numbers
- model numbers
- exact dimensions
- chemical names
- technical terminology
- uncommon product codes

---

## 5. Structured retrieval

Structured filtering handles deterministic constraints.

Example:

> "10mm hex bolt under $20"

Query compiler:

```text
product_type = bolt
AND diameter = 10mm
AND head_type = hex
AND price <= 20
```

This should be resolved through structured indexes, not semantic similarity.

---

## 6. Fine-grained semantic attributes

For:

> "ribbed silk shirt"

the representation becomes:

```text
product_type = shirt
material = silk
texture = ribbed
```

The system should separately model:

```text
MATERIAL
  silk
  cotton
  satin
  wool

TEXTURE
  ribbed
  smooth
  brushed
  knitted
```

This prevents semantic collapse between visually similar but materially different products.

---

## 7. Candidate generation

Target:

```text
BM25             → 300–500
Dense ANN        → 300–500
Structured       → 100–300
                     ↓
              Deduplicate
                     ↓
              300–800 candidates
                     ↓
              Lightweight LTR
                     ↓
                   Top-K
```

ANN should use approximate search with tuned `efSearch`/probe parameters to meet the latency target.

---

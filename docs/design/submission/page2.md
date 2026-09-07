# Cold-Start Discovery: Turning Silent Products into Searchable Products

## 1. The fundamental problem

A product such as:

```text
SKU: 89372
Category: Women's Dress
Brand: X
Price: $49
Image: [dress photograph]
Video: [10-sec video]
```

has almost no useful text.

A query:

> "summer beach party dress"

cannot reliably match this product through BM25 because the words do not exist.

The solution is to **materialize semantic information from the visual assets offline.**

---

## 2. Multi-stage enrichment pipeline

```text
                   PRODUCT INGESTION
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
         Metadata      Images       Video
             │           │           │
             │           ↓           ↓
             │       Image Quality   Keyframes
             │           │           │
             │           ↓           ↓
             │       OCR / Objects / Visual
             │           │           │
             └───────────┼───────────┘
                         ↓
                 HEAVY VLM / LLM
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Captioning     Attributes      Use Cases
          │              │              │
          ↓              ↓              ↓
       Style/Vibe    Materials       Compatibility
          │              │              │
          └──────────────┼──────────────┘
                         ↓
               CANONICAL PRODUCT MODEL
                         ↓
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Text Index     Vector Index   Structured
                                      Index
```

---

## 3. Four enrichment levels

### L0 — Raw representation

```text
SKU
Brand
Category
Price
Images
Video
```

### L1 — Perceptual representation

Extract what is visibly present:

- Objects
- Colors
- Shapes
- Material cues
- Texture
- OCR
- Logo
- Packaging
- Dimensions
- Visual attributes

### L2 — Semantic representation

Convert perception into language:

```json
{
  "caption": "Women's floral maxi dress suitable for beach occasions",
  "style": ["bohemian", "summer", "casual"],
  "occasion": ["beach", "vacation", "party"],
  "materials": ["silk"],
  "texture": ["ribbed"],
  "visual_attributes": ["floral", "long sleeve"]
}
```

### L3 — Retrieval representation

Generate:

- Product semantic text
- Dense text embedding
- Image embedding
- Video embedding
- Attribute embeddings
- Sparse/BM25 representation
- Structured attribute index

---

## 4. Heavy model → lightweight production model

Use foundation models primarily during enrichment.

```text
             SOTA VLM / LLM
                    │
                    │ generates
                    ↓
       High-quality semantic labels
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Caption   Attribute   Embedding
                    │
                    ↓
              Training Data
                    │
                    ↓
        Domain-specific Student
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Encoder   Classifier  Attribute
                              model
                    │
                    ↓
               ONLINE PATH
```

This is substantially cheaper than putting a large VLM in every search request.

---

## 5. TSR / complex table handling

Some product information should **not be flattened into ordinary text**.

Examples:

- Size chart
- Nutrition label
- Technical specification table
- Compatibility table
- Dimensions
- Product variants

Pipeline:

```text
Image / PDF
    │
    ↓
OCR
    │
    ↓
Layout Detection
    │
    ↓
Table Structure Recognition
    │
    ↓
Canonical Table JSON
    │
    ├──────────────→ Structured Filter
    │
    └──────────────→ Table Text / Sparse Index
```

Example:

```json
{
  "table_type": "technical_specification",
  "rows": [
    {
      "attribute": "diameter",
      "value": 10,
      "unit": "mm"
    },
    {
      "attribute": "head_type",
      "value": "hex"
    }
  ]
}
```

This is what makes **"10mm hex bolt"** reliably searchable.

---

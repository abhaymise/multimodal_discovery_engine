# Multimodal Alignment & Model Strategy

## 1. The central ML problem

We need the following to be close in a shared semantic space:

```text
"summer beach party"
          │
          ↕
      latent space
          ↕
 image of floral beach dress
```

The user vocabulary does not need to exist literally in the product catalog.

---

## 2. Multimodal representation architecture

Use a strong pretrained vision-language representation model as the starting point.

Candidate model families:

- SigLIP / SigLIP2
- CLIP-family models
- Modern vision-language encoders
- Domain-adapted multimodal encoders

Large VLMs such as Qwen-VL-class models are better suited to **offline semantic understanding** than serving every query.

### Important distinction

```text
VLM
=
semantic understanding

Multimodal encoder
=
retrieval representation
```

Do not automatically use the same model for both.

---

## 3. Offline teacher / online student

```text
                       RAW PRODUCT
                           │
                           ↓
                  LARGE VLM / LLM
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Caption      Attributes       Concepts
             │             │             │
             └─────────────┼─────────────┘
                           ↓
                    Teacher Signals
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
     Contrastive       Distillation      Classification
       targets           targets            targets
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                DOMAIN-SPECIFIC ENCODER
                           │
                           ↓
                    ONNX / INT8
                           │
                           ↓
                    ONLINE SEARCH
```

---

## 4. Training strategy

### Stage 1 — Foundation model initialization

Start from a strong pretrained multimodal encoder.

### Stage 2 — Domain adaptation

Train on marketplace data:

```text
(query, positive product)
(query, hard negative product)
(text, image)
(image, image)
(attribute, product)
```

### Stage 3 — Teacher distillation

Teacher provides:

- semantic similarity
- attribute labels
- visual concepts
- captions
- hard-negative relationships

Student learns to reproduce these signals.

### Stage 4 — Retrieval optimization

Optimize directly for:

- Recall@K
- NDCG
- cross-modal retrieval
- hard-negative separation

---

## 5. Contrastive learning

For a query \(q\) and positive product \(p^+\):

\[
s(q,p)=
\frac{E_q(q)^T E_p(p)}
{\|E_q(q)\|\|E_p(p)\|}
\]

Train using temperature-scaled contrastive loss:

\[
L =
-\log
\frac{
e^{s(q,p^+)/\tau}
}{
\sum_j e^{s(q,p_j)/\tau}
}
\]

Hard negatives are particularly important:

```text
"red silk shirt"
       │
       ├── positive: red silk shirt
       ├── hard negative: red satin shirt
       ├── hard negative: red cotton shirt
       └── hard negative: blue silk shirt
```

---

## 6. Why this solves subjective queries

Suppose the catalog contains:

```text
"floral maxi dress"
```

but not:

```text
"summer beach party"
```

The offline VLM can infer:

```text
floral
summer
beach
vacation
party
bohemian
```

The shared embedding space then allows:

```text
"summer beach party"
          ↓
multimodal embedding
          ↓
ANN
          ↓
products visually/semantically aligned
```

This is **semantic generalization**, not keyword matching.

---

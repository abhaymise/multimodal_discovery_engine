# Multimodal Discovery Engine
### Scalable semantic discovery platform 

- Supports text, image, and video search 
- Designed to `scale` across 5+ millions of products and catalogs, 
 with sub-120ms P99 `latency` and <$5K/month online GPU/compute `cost`.
- Supports multimodal search (text → image, image → product, video → product) and cross-modal search (text → video, image → product, video → product). 
- Support subjective search (e.g. "summer beach party") and fine grained objective search (e.g. "10mm stainless steel hex bolt") with high accuracy.
- Supports hybrid retrieval 
  - Dense retrieval (ANN) for semantic search
  - Lexical retrieval (BM25) for technical search
  - Attribute-based retrieval/filtering for structured search
- Designed using multi stage funnel retrieval architecture for low-latency search across large catalogs.
- Supports easy onboarding of new catalogs and products with minimal human intervention, using comprehensive multimodal metadata enrichment pipelines.
- Designed on philosophy of using heavy models for offline enrichment and light models for online retrieval, to reduce online compute cost while maintaining high accuracy.

# Solution Design Documentation

This documentation is split into several files for easier reading.

- [Problem, Constraints & Architecture Thesis](./page1.md)
- [Cold-Start Catalog Enrichment](./page2.md)
- [Canonical Product Representation & Feature Store](./page3.md)
- [Multimodal Alignment & Model Strategy](./page4.md)
- [Query Understanding + Hybrid Retrieval](./page5.md)
- [Vibe Search, Image/Video Search & Ranking](./page6.md)
- [Production Architecture, Latency, Cost & Evaluation](./page7.md)






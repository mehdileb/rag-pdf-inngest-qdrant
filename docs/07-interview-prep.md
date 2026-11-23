# 07 — Interview preparation (AI & Data Engineering PFE)

## Five questions and concise answer guides

**1. Trace a PDF from Streamlit to Qdrant.**
`UploadedFile` → save to disk → `rag/ingest_pdf` event (path, source_id) → Inngest trigger → `_load` uses PDFReader + SentenceSplitter → `_upsert` uses embeddings, deterministic UUIDs and Qdrant payloads → upsert points.

**2. What is the difference between an event payload and a Qdrant payload?**
Inngest event `data` transports workflow inputs (`pdf_path`, `source_id`, or `question`, `top_k`). Qdrant payload stores per-vector metadata and original text (`source`, `text`). Created in Streamlit and `main.py` respectively.

**3. Why not embed the entire PDF as a single vector?**
A single vector loses passage-level retrieval resolution. Chunking balances granularity, context, indexing cost and answer quality. Overlap protects boundary context but creates duplicate text.

**4. Why are both `text-embedding-3-large` and GPT-4o mini used?**
Different jobs: encode text into searchable vectors vs synthesize an answer from retrieved text. They do not need to be “paired” by brand. Index/query embeddings need the same compatible space.

**5. What would you change for a real multi-client service?**
Tenant-based auth and payload filters, robust uploads/OCR, document IDs and versioning, timeouts, observability, relevance thresholds, citations and a measured evaluation suite.

## Python exercises (solve independently)

1. Convert `["A", "B"]` into `[{"text": "A", "source": "doc.pdf"}, ...]` with a list comprehension.
2. Given `responses = [[0.1, 0.2], [0.4, 0.5]]`, explain `responses[0]` vs `responses[0][0]`.
3. Implement `unique_sources(hits: list[dict]) -> list[str]` retaining first-seen order, unlike a plain `set`.
4. Add `try/except` around a hypothetical embedding API call. Distinguish network retryable errors and invalid input errors.
5. Explain why `lambda: _load(ctx)` is passed into `ctx.step.run` and what `await` does.

## Discussion prompts

- Which aspects did you personally code versus only inspect?
- Why are vector similarity and factual correctness different?
- How would you measure retrieval quality (Recall@K, MRR) and answer groundedness?
- When would you choose simpler synchronous FastAPI over event workflows?
- What if the embedding provider is down, or a PDF is image-only?

**Be transparent:** this was a guided code study. Cite independently implemented extensions only after they exist and are tested.

# 06 — Engineering review: what is and is not production-ready

These are **observed limitations and proposed improvements**, not claims of implemented functionality.

| Observed in supplied code | Risk | Proposed improvement |
|---|---|---|
| File paths in Inngest event | Backend on another host may not access upload | Object storage / shared storage with durable URI |
| Filename-based source ID | Collisions; same filename with different content | Document UUID + content hash + tenant ID |
| Deterministic UUID from source and chunk index | Old chunks may remain if replacement has fewer chunks | Versioning and cleanup of stale points |
| No OCR in PDF reader pipeline | Scanned PDFs can yield empty text | OCR fallback and scanned-PDF detection |
| No file validation / name sanitization shown | Invalid uploads, overwrites, unsafe paths | MIME, file size, filename sanitization, unique storage name |
| Search across `docs` with no metadata filters | Cross-document/tenant retrieval | Tenant/user auth and mandatory query filters |
| `search()` discards scores | Cannot apply threshold; weaker observability | Return `(text, source, score, page)` for each hit |
| `sources = set()` | Text/source alignment and order lost | Preserve each hit's provenance in a record |
| No no-answer guard in prompt | Unsupported answers | Evaluate relevance, explicit abstention rule, citations |
| Upload event sent on Streamlit rerun | Duplicate processing possible | Explicit Index button + session-state job tracking |
| `requests.get` lacks explicit timeout | UI can hang during polling | Request timeouts, structured errors, backoff |
| No tests demonstrated | Reliability not verified | Unit tests + integration tests + evaluation set |
| No deployment configuration beyond local dev shown | Production readiness not verified | Secrets handling, containers, health checks, monitoring |

## Model/cost decision: accurate interview framing

- `text-embedding-3-large` is a paid API embedding model, defaulting to 3072 dimensions. The tutorial **chooses** it; the archive provides no evaluation proving it is best for the specific PDFs.
- `gpt-4o-mini` is a separate, paid generation model. **The two are not inherently coupled.** There is no rule that `text-embedding-3-large` is the embedding model that “works with GPT-4o mini best.”
- To lower costs, benchmark `text-embedding-3-small`, local sentence-transformer encoders and different generation models, measuring retrieval recall/precision, answer groundedness, latency and cost on representative questions.
- For an alternate embedding model, recompute existing document vectors and ensure its dimension is reflected in Qdrant's collection config. Changing generation models generally does not require re-embedding by itself.
- Never say you sold the project, performed client trials, or measured optimality unless you can substantiate those facts.

## Candidate's potential independent extension roadmap

1. Add content-hash IDs and `tenant_id` to payloads.
2. Implement a typed `/documents` API / job status endpoint with FastAPI.
3. Add OCR fallback for image-only PDFs.
4. Return scored, page-cited search hits and document-filtered retrieval.
5. Test on a small labeled Q&A dataset; compare cost/quality across embedding models.
6. Deploy with secrets, object storage and observability; document reproducible runs.

**Status:** proposals only — not implemented in the supplied code.

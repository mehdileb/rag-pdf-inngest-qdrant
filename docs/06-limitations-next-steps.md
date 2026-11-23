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



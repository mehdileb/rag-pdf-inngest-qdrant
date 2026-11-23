# 08 — Setup notes and verification checklist

This document describes the **observed code prerequisites**; we did not execute or validate the complete system during documentation preparation.

## Runtime clues from `pyproject.toml`

- `requires-python = ">=3.13"`
- Dependencies include `fastapi`, `inngest`, `llama-index-core`, `llama-index-readers-file`, `openai`, `python-dotenv`, `qdrant-client`, `streamlit` and `uvicorn`.
- Local defaults: Qdrant on `http://localhost:6333`; Inngest dev API `http://127.0.0.1:8288/v1`; FastAPI server address is whatever Uvicorn is configured to serve.
- OpenAI auth is read from `OPENAI_API_KEY`.

## Indicative local startup sequence (adapt to installed CLI versions)

1. Create a Python 3.13+ environment and install the dependencies declared by the tutorial.
2. Start a Qdrant server listening on port 6333.
3. Set `OPENAI_API_KEY` securely via environment variables; do not commit `.env`.
4. Start the FastAPI app, e.g. `uvicorn main:app --reload` (default Uvicorn port 8000).
5. Start the Inngest development server configured to register/sync the FastAPI Inngest endpoint (verify your Inngest CLI version and endpoint path in its docs).
6. Start the UI, e.g. `streamlit run streamlit_app.py`.
7. Upload a **text-bearing** PDF; confirm the Inngest run completed and Qdrant has points.
8. Ask a question; confirm Qdrant retrieval, completed query run, displayed answer and source.

## Verification scenarios (not yet executed)

- One-page digital PDF with a known fact; ask for that fact.
- Image-only scanned PDF; verify extraction limitation and OCR backlog.
- Same filename uploaded twice; inspect source-ID collision, rate limiting and duplicate behavior.
- A question not answered by any PDF; verify whether model abstains or hallucinates.
- Simulate API/network errors to confirm proper UI and retry behavior.
- Check multi-document retrieval and absence of tenant isolation.

## Security and licensing

Do not commit `.env`, credentials, `uploads/` or `qdrant_storage/` contents. Ensure `.gitignore` includes these paths. Confirm the original tutorial code's license before republishing it: this documentation pack does **not** grant redistribution rights to third-party code.

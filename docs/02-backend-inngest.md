# 02 — FastAPI, Inngest, workflow steps and Pydantic

## What is an endpoint?

An HTTP endpoint is an address plus method (for example `GET /documents`). FastAPI is a Python backend framework. Here the project does **not** define a custom `@app.post("/upload")` handler: `inngest.fast_api.serve(app, inngest_client, [rag_ingest_pdf, rag_query_pdf_ai])` mounts the Inngest HTTP integration on the FastAPI app. Network/event communication exists despite no user-defined REST upload route.

## What does the SDK do?

The Inngest Python SDK wraps API and workflow integration details. `is_production=False` selects development-mode behavior; it **does not** make the full RAG application offline. OpenAI calls still require remote API access.

## What does the decorator do?

`@inngest_client.create_function(...)` **registers/configures** an Inngest-invoked function. `trigger=inngest.TriggerEvent(event="rag/ingest_pdf")` associates it with the event name. It is not the history of previous function runs.

### Throttle vs rate_limit

The ingestion decorator declares `Throttle(count=2, period=1 minute)` and `RateLimit(limit=1, period=4 hours, key="event.data.source_id")`. They govern execution pace; the second uses the supplied source ID as its key. These controls are not a permanent content-hash-based deduplication mechanism, nor inherently per-user authentication. The finer scheduler semantics are SDK/version-dependent.

## Who creates `ctx`?

The workflow is invoked by Inngest and receives a context object (`ctx: inngest.Context`), which exposes e.g. `ctx.event.data` and `ctx.step`. Streamlit never needs to create that object itself.

Inside the ingestion workflow, `_load(ctx)` **receives** `ctx` through `lambda: _load(ctx)` passed to `ctx.step.run`. The two `ctx` names refer to the same provided object for this call, but are parameters in different function scopes.

## Two ingestion steps

1. `ctx.step.run("load-and-chunk", lambda: _load(ctx), output_type=RAGChunkAndSrc)` reads `pdf_path`, calls `load_and_chunk_pdf`, returns Pydantic `RAGChunkAndSrc(chunks=..., source_id=...)`.
2. `ctx.step.run("embed-and-upsert", lambda: _upsert(chunks_and_src), output_type=RAGUpsertResult)` embeds text, builds IDs and payloads, inserts into Qdrant and returns the **number of chunks**.

`lambda` gives the orchestrator a callable; `_load(ctx)` without `lambda` would invoke it immediately during argument evaluation. `await` waits for the async step call.

## Pydantic vs payload

Pydantic defines typed data models (`RAGChunkAndSrc`, `RAGUpsertResult`, `RAGSearchResult`, `RAQQueryResult`); payload means data transported by an event (`Event.data`) **or** metadata stored with a Qdrant point. A Pydantic model is not automatically the original event payload. `model_dump()` converts a Pydantic model into a Python dict.

### Concrete result

If `load_and_chunk_pdf` returns three chunks, `_load` produces an object with `chunks` and `source_id`, and `_upsert` returns `RAGUpsertResult(ingested=3)`, not a text-length measurement.

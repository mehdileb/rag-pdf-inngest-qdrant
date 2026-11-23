# 04 — Qdrant point storage and semantic retrieval

## `QdrantStorage.__init__`

`QdrantClient(url="http://localhost:6333", timeout=30)` configures access to the local Qdrant HTTP server. `self.collection = "docs"`; if missing, the code creates it with `VectorParams(size=3072, distance=Distance.COSINE)`.

An existing collection's embedding dimension / distance compatibility is **not validated** by the constructor.

## `upsert(ids, vectors, payloads)`

The code builds `PointStruct(id=ids[i], vector=vectors[i], payload=payloads[i])` for each index `i`, then calls `self.client.upsert(...)`.

Crucially the Qdrant payloads originate in **`main.py` `_upsert`**, NOT Streamlit:

```python
payloads = [{"source": source_id, "text": chunks[i]} for i in range(len(chunks))]
```

A Qdrant point = ID + vector + payload; the payload holds the **original text** for later LLM context. A deterministic ID uses `uuid.uuid5(uuid.NAMESPACE_URL, f"{source_id}:{i}")`. Upsert inserts or replaces by ID, not by semantic duplicate detection.

## `search(query_vector, top_k=5)`

The examined code calls:

```python
results = self.client.search(
    collection_name=self.collection,
    query_vector=query_vector,
    with_payload=True,
    limit=top_k,
)
```

Qdrant retrieves up to `top_k` similar points using the collection's cosine configuration. `with_payload=True` is essential here to recover the actual passage texts. The code loops over results and builds `contexts = []` (ordered text list) and `sources = set()` (deduplicated source filenames). The returned Python dict has keys `contexts` and `sources`.

### Cosine similarity

For nonzero vectors `a`, `b`: `cosine(a, b) = (a · b) / (||a|| ||b||)`. It measures directional similarity, not factual truth. A passage about **revenue in 2024** can be very close semantically to a question about **revenue in 2025** yet be the wrong answer.

### top_k trade-off

Low `top_k` may miss supporting facts; high `top_k` may increase noise and token costs. The UI defaults to 5 and allows 1–20. The original implementation does not return relevance scores or filter by document/tenant. These are engineering limitations, not implemented capabilities.

**API compatibility note:** this repository uses `QdrantClient.search`; consult the installed `qdrant-client` version if upgrading to newer query APIs such as `query_points`.

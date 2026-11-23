# 03 — PDF processing, chunking and embedding

## Exact implementation in `data_loader.py`

```python
client = OpenAI()
EMBED_MODEL = "text-embedding-3-large"
EMBED_DIM = 3072
splitter = SentenceSplitter(chunk_size=1000, chunk_overlap=200)
```

`load_dotenv()` loads environment variables before the OpenAI client is constructed. The `EMBED_DIM` constant is defined but not directly referenced in this file; `vector_db.py` independently defaults to 3072.

### Reading the PDF

`docs = PDFReader().load_data(file=path)` returns document objects. `texts = [d.text for d in docs if getattr(d, "text", None)]` extracts non-empty textual content. **This does not create a robust OCR pipeline.** Scanned, image-only PDFs may not yield useful text.

### Chunking

`splitter.split_text(t)` divides text using the LlamaIndex sentence-aware splitter, configured for a target size of 1000 tokens and an overlap of 200 tokens. The implementation loops through the extracted text and calls `chunks.extend(...)` to add each resulting chunk; `extend` adds list elements individually, unlike `append` which would create nested lists when given a list.

Why chunk? A full PDF represented as a single vector loses passage-level granularity. Too-small chunks risk losing context; too-large chunks risk retrieving irrelevant material. Larger overlap improves boundary continuity but increases duplication and embedding costs. The numbers 1000/200 were chosen in the tutorial; **no benchmark demonstrating optimality is provided**.

### Embeddings

```python
def embed_texts(texts: list[str]) -> list[list[float]]:
    response = client.embeddings.create(model=EMBED_MODEL, input=texts)
    return [item.embedding for item in response.data]
```

For N input strings, a successful request normally returns N embeddings. The result is a nested list: one list of 3072 floats per input string in this implementation. `embed_texts([question])[0]` converts a single question to a list input, then retrieves the first **whole** vector; it is **not** selecting dimension zero.

The embedding model is pretrained. It converts the supplied text to a semantic vector; adding PDFs to Qdrant does not fine-tune model weights. Qdrant **does not generate embeddings** in this implementation.

### Why use the same embedding model at index and query time?

Similarity retrieval assumes compatible representations in the same vector space. Switching embedding models after indexing generally requires recomputing all document embeddings and ensuring the collection dimensions match. The embedding model need **not** be a special partner of GPT-4o mini; generation and embedding are separate operations.

# 05 — Prompt construction, LLM inference and result display

## Query step inside `main.py`

`rag_query_pdf_ai(ctx)` is triggered by `rag/query_pdf_ai`. It reads `question` and `top_k` from `ctx.event.data`, then calls an orchestrated `embed-and-search` step. `_search` embeds `[question]`, selects vector `[0]`, queries Qdrant and returns typed `RAGSearchResult(contexts=..., sources=...)`.

## Build the prompt from retrieved **text**, not vectors

```python
context_block = "

".join(f"- {c}" for c in found.contexts)
user_content = (
    "Use the following context to answer the question.

"
    f"Context:
{context_block}

"
    f"Question: {question}
"
    "Answer concisely using the context above."
)
```

The list of retrieved passages is joined into one string. The instruction to use context does not itself guarantee groundedness; retrieved content can be irrelevant or malicious.

## Inference

The project creates `ai.openai.Adapter(auth_key=os.getenv("OPENAI_API_KEY"), model="gpt-4o-mini")` and calls `await ctx.step.ai.infer("llm-answer", adapter=adapter, body={...})` with `max_tokens=1024`, `temperature=0.2`, a system message and the user message containing context + question.

The code extracts `res["choices"][0]["message"]["content"].strip()` and returns `answer`, `sources`, `num_contexts` as a dict. Streamlit polls the Inngest dev API for the completed run and displays the result.

## Key distinctions

- Embedding model: maps input text to vectors for retrieval.
- Vector database: retrieves source passages.
- Generative LLM: reads the **source passages as text** in a prompt and composes an answer.
- RAG does **not** fine-tune or retrain GPT-4o mini.
- The current `sources` output identifies files, not precise page-level citations.

## Failure case

If only 2024 revenue is retrieved for a 2025 question, a high vector score does not make the 2024 figure a valid answer. Improved systems use metadata constraints (e.g. year), reranking, validation and an explicit "insufficient evidence" response.

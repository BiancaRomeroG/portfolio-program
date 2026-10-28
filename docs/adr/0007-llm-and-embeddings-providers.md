# ADR-0007: LLM and embeddings providers for incident-copilot

- Status: accepted
- Date: 2026-10-28

## Decision
- Abstraction: Vercel AI SDK (`ai`). The model is created by `createModel()` in `apps/api/src/ai/provider.ts` based on `AI_PROVIDER` env: `groq` (default, `@ai-sdk/groq`, model `llama-3.3-70b-versatile` or newer free model) or `ollama` (`ollama-ai-provider`, model `qwen2.5:7b` or `llama3.1:8b`). Adding `gemini` later is one file.
- Embeddings: `@huggingface/transformers` (transformers.js) in-process with `Xenova/bge-small-en-v1.5` (384 dims, quantized). Model files are cached in a Docker volume / Render disk; first boot downloads ~30 MB. No external key.
- Vector store: Postgres + pgvector (Supabase). HNSW index. Hybrid retrieval = pgvector cosine + `tsvector` full text, fused with Reciprocal Rank Fusion. Optional cross-encoder rerank with `Xenova/ms-marco-MiniLM-L-6-v2` in-process.
- LLM-as-judge for evals also runs on Groq; evals run nightly, not on every PR, to respect free rate limits.

## Consequences
- Hosted demo works with a free Groq key; offline dev works with Ollama.
- Embedding dimension is fixed at 384; changing the model requires a re-index job (documented in runbook).
- Rate limits on Groq free tier: add retry with backoff and a visible "rate limited, retrying" state in the UI.

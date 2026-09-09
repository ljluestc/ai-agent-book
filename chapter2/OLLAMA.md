# Running Chapter 2 examples against a local Ollama (no API key)

Every LLM-backed Chapter 2 example resolves its provider through the shared
`agentbook.providers` registry, which already includes a **keyless** `ollama`
backend (`http://localhost:11434/v1`, default model `qwen3:8b`,
`requires_key=False`). So you can run the examples entirely locally — no cloud
key, no spend.

## 1. Start Ollama and pull a model

```bash
ollama serve                      # or run the ollama container/pod
ollama pull qwen3:8b              # or a smaller model, e.g. qwen2.5:0.5b
```

## 2. Point an example at it

```bash
export LLM_PROVIDER=ollama
# optional overrides:
export OLLAMA_BASE_URL=http://localhost:11434/v1   # e.g. an in-cluster Service
export MODEL_NAME=qwen2.5:0.5b                      # match what you pulled
```

Then run any example as usual, e.g.:

```bash
# context-compression (2-10): single-strategy smoke
python context-compression/main.py --strategy summary

# system-hint (2-9): the CLI now accepts --provider ollama with no key
python system-hint/main.py --mode single --provider ollama --task "list the files here"

# kv-cache (2-3): agent resolves any registered provider
python kv-cache/main.py --provider ollama ...

# prompt-injection (2-5): OpenAI-compatible, point --base_url at Ollama
python prompt-injection/demo.py --base_url http://localhost:11434/v1 --model qwen3:8b
```

## Notes

- `local_llm_serving` (2-1) already targets Ollama/vLLM natively — nothing to change.
- `attention_visualization` (2-2/2-8) loads local **transformers** weights to plot
  attention; Ollama does not expose attention tensors, so that one is not an
  Ollama target by design.
- Small models (0.5B–3B) may do tool-calling less reliably than the cloud
  defaults; use them for wiring/smoke verification, and a larger local model
  (or a cloud provider key) when reproducing the book's quality numbers.
- In-cluster: set `OLLAMA_BASE_URL` to the Ollama Service DNS, e.g.
  `http://ollama.<namespace>.svc:11434/v1`.

# The `keep_alive=0` startup-hang problem

## Symptom
A model-backed UI shows its logo / splash and appears to **hang on startup**.
The host itself is fine and the app does not crash — it just sits on the logo.

## What it is *not*
- **Not an OS boot hang.** The machine stays up for weeks; a boot hang would reset uptime.
- **Not a crash.** No panic / crash / hang diagnostic reports are produced — the
  process stalls, it does not die.
- **Not the web layer.** The UI's own health endpoint responds in milliseconds.

## What it actually is
The stall is **model cold-load on first use**.

With `keep_alive=0`, Ollama **evicts every model from RAM immediately after each
use**. So the loaded-model set is empty between requests, and *every* new chat
pays a full cold-load cost. On a RAM-constrained host — especially with
interactive chat and a review job competing for memory at the same time — that
first-token wait can stretch to tens of seconds and reads as a hang at the logo.

```
UI logo shown ──> first request ──> Ollama cold-loads model into RAM ──> first token
                                     ^ (seconds under memory pressure) = looks hung
```

## Why `keep_alive=0` is set on purpose
It is a deliberate memory-protection policy. The review chain runs **sequentially**
(one model at a time) under a tight RAM ceiling; a lingering model would crowd out
the next stage. `keep_alive=0` guarantees each stage starts from a known-free state.
It pairs with the other bounded local-call limits (`num_ctx=4096`,
`max_input_tokens≤3500`, `num_predict≤300`, `request_timeout=240s`).

## The tradeoff (owner decision)
A nonzero `OLLAMA_KEEP_ALIVE` keeps a default model warm and removes the
per-open cold load — but it **permanently pins that model in RAM**, directly
against the ceiling that `keep_alive=0` exists to protect. You cannot have both
"instant first token" and "no resident model" on a tight-memory host; pick per use.

## Cheaper mitigations (no policy change)
- Use a **smaller default model** for the interactive UI → faster cold-load.
- **Don't run review jobs concurrently** with interactive chat (the memory collision).
- **Pre-warm** the model immediately before an interactive session, accepting it will
  be evicted after.
- **Measure, don't guess:** watch the loaded-model set and time-to-first-token to
  confirm cold-load is the cause before changing anything.

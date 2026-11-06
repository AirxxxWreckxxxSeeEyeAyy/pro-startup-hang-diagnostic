# Air–Pro ChainOps — Keep-Alive Notes

Public-safe guide to the local-model review chain and the Ollama `keep_alive=0`
startup-hang problem.

These are **conceptual guides only.** Per project policy, host descriptors, IPs,
keys, private manifests, runtime reports and full model responses are kept out
of version control.

## Contents
- [Local-model review chain](docs/local-model-chain.md) — Eco / Standard / Intensive modes and the sequential Qwen → Llama → Gemma review (context for the problem below).
- [The `keep_alive=0` startup-hang problem](docs/keep-alive-problem.md) — why a model-backed UI can appear to hang on its logo, and the tradeoff.

## Already documented elsewhere
The **root-task planning flow** and the **gated weekly publication** pipeline are
maintained in the `air-pro-chainops` repository and are not duplicated here:
`prompts/root-task.md`, `task_flow.json`, `schemas/root-task.schema.json`,
`docs/SCHEDULE.md`, `docs/PUBLISHING.md`, `publish_weekly.py`.

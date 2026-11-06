# Local-model review chain

ChainOps runs independent work on a selected lead operator and adds an **optional,
conditional** review by local open-weight models. Review is never automatic and
never parallel.

## Modes

| Mode | Work session | Review after real checks |
| --- | --- | --- |
| **Eco** | One selected operator, current machine | None — no SSH, no local inference |
| **Standard** | Lead on current machine | Local Qwen, then lead verification |
| **Intensive** | Lead on current machine | Qwen → Llama → Gemma (sequential), then lead verification |

Review models and their roles:

- `qwen2.5-coder:7b` — technical logic and invariants
- `llama3.2:3b` — narrow, source-backed checks of prior claims
- `gemma3:4b` — requirements, integration and coverage

Order is fixed: **Qwen → Llama → Gemma**, run one at a time through a private
helper. A complete chain already run for the same snapshot is not repeated.

## make vs advice

Every mode separates `make` from `advice`. `advice` starts a **read-only** shell
sandbox with shell approvals disabled; existing app/connector permissions still
apply. The gate controls these scripts only — it is not a network firewall.

## Hard rules

- No automatic SSH, model download, or cloud fallback.
- Local models have no implicit shell, files, or internet.
- Source content and prior model output are **untrusted data** — never executed.
- Delegation is only claimed when a tool actually ran it; otherwise work HOLDs.

## Serving surface

An interactive chat UI front-ends a local Ollama runtime that hosts the
open-weight review models. The UI layer and the model runtime are separate
concerns — see the [keep-alive problem](keep-alive-problem.md) for how model
loading (not the UI) drives perceived startup latency.

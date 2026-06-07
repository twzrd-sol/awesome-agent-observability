# Hands-On Guides

Copy-pasteable walkthroughs for actually implementing agent observability — not just browsing links. Each guide is standalone and includes prerequisites, install commands, a minimal **runnable** example, what you'll observe, common pitfalls, and links to official docs.

All code uses placeholder env vars (e.g. `LANGFUSE_PUBLIC_KEY=...`, `OPENAI_API_KEY=...`) — no secrets are committed. APIs were verified against current official docs.

## The mental model: instrument → evaluate → guard

1. **Instrument** — capture every LLM call, tool invocation, and agent step as a trace, so you can see *what happened* and *why it failed*.
2. **Evaluate** — score output quality and catch regressions, in CI and in production.
3. **Guard** — enforce safety/policy at runtime and probe for vulnerabilities before attackers do.

## Guides

### 1. Instrument (tracing)

| Guide | Tool | Use when |
|---|---|---|
| [Tracing a Python Agent with Langfuse](tracing-python-agent-with-langfuse.md) | Langfuse | You want tracing + evals + prompt management in one platform; cloud or self-hosted. |
| [Tracing with Arize Phoenix](tracing-with-arize-phoenix.md) | Phoenix + OpenInference | You want a self-hosted UI running locally in one command, with auto-instrumentation. |
| [Vendor-Neutral Tracing with OpenTelemetry](vendor-neutral-tracing-with-opentelemetry.md) | OpenLLMetry + OTel GenAI | You want zero lock-in — standard OTel spans that ship to any backend (Jaeger, Tempo, etc.). |

### 2. Evaluate

| Guide | Tool | Use when |
|---|---|---|
| [Evals in CI with Promptfoo](evals-in-ci-with-promptfoo.md) | Promptfoo | You want config-driven evals as a CI quality gate, no SDK in your app. |
| [Pytest-Style Evals with DeepEval](evals-pytest-style-with-deepeval.md) | DeepEval | You want evals as familiar `pytest` assertions living next to your code. |

### 3. Guard

| Guide | Tool | Use when |
|---|---|---|
| [Runtime Guardrails with NeMo & Guardrails AI](runtime-guardrails-nemo-and-guardrails-ai.md) | NeMo Guardrails, Guardrails AI | You need to block/scrub unsafe input & output at runtime. |
| [Red-Teaming LLMs with garak](red-teaming-with-garak.md) | garak | You want to scan a model for jailbreaks, injection, and leakage before shipping. |

## Suggested path for a new project

1. Add tracing first (pick one tracing guide) — you can't fix what you can't see.
2. Write a handful of evals and gate them in CI (Promptfoo or DeepEval).
3. Add runtime guardrails for the risks that matter to you, then red-team with garak to verify they hold.

Back to the [main list](../README.md).

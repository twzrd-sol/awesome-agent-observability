# Awesome Agent Observability [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of tools for observability, evaluation, tracing, and guardrails of production LLM agents.

Shipping an agent to production is the easy part. Knowing *why* it failed a tool call, regressed after a prompt change, or leaked a secret is the hard part. This list collects the tools, frameworks, and standards that make agentic systems measurable, debuggable, and safe — the "ops" layer that most agent tutorials skip.

Scope: tools focused on **tracing, evaluation, monitoring, and guardrails for LLM agents**. Generic APM and broad LLMOps catalogs are out of scope unless they have first-class agent support.

> **New:** This list now ships with [**hands-on implementation guides**](guides/) — copy-pasteable walkthroughs that take you from `pip install` to a working traced, evaluated, and guarded agent. Browse the index below or jump straight to the [guides](guides/README.md).

## Contents

- [How to Use This List](#how-to-use-this-list)
- [Hands-On Guides](#hands-on-guides)
- [Tracing & Observability Platforms](#tracing--observability-platforms)
- [Evaluation Frameworks](#evaluation-frameworks)
- [Guardrails & Safety](#guardrails--safety)
- [Red Teaming & Security Testing](#red-teaming--security-testing)
- [Instrumentation & Standards](#instrumentation--standards)
- [Benchmarks](#benchmarks)
- [Gateways with Built-in Observability](#gateways-with-built-in-observability)
- [Related Lists](#related-lists)

## How to Use This List

Production agent observability comes down to three steps. Do them in order — you can't evaluate what you can't see, and you can't guard what you haven't measured.

1. **Instrument** → capture every LLM call, tool invocation, and agent step as a trace. Start here. Reach for a [tracing platform](#tracing--observability-platforms): pick **Langfuse** for an all-in-one open-source platform, **Arize Phoenix** for a local-first UI in one command, or **plain OpenTelemetry + OpenLLMetry** if you want zero vendor lock-in.
2. **Evaluate** → score output quality and catch regressions in CI and in production. Reach for an [eval framework](#evaluation-frameworks): **Promptfoo** for config-driven CI gates, **DeepEval** for `pytest`-style assertions, **Ragas** for RAG-specific metrics.
3. **Guard** → enforce safety/policy at runtime and probe for vulnerabilities before attackers do. Reach for [guardrails](#guardrails--safety) (**Guardrails AI**, **NeMo Guardrails**) and [red-teaming](#red-teaming--security-testing) (**garak**).

**Standards tie it together:** if you care about portability, prefer tools built on the [OpenTelemetry GenAI conventions](#instrumentation--standards) so your traces flow between vendors without rewrites.

## Hands-On Guides

Concrete, runnable walkthroughs live in [`guides/`](guides/README.md). Each includes prerequisites, install commands, a minimal working code example, what you'll observe, and common pitfalls.

| Guide | Step | Tool |
|---|---|---|
| [Tracing a Python agent with Langfuse](guides/tracing-python-agent-with-langfuse.md) | Instrument | Langfuse |
| [Tracing with Arize Phoenix](guides/tracing-with-arize-phoenix.md) | Instrument | Phoenix + OpenInference |
| [Vendor-neutral tracing with OpenTelemetry](guides/vendor-neutral-tracing-with-opentelemetry.md) | Instrument | OpenLLMetry + OTel GenAI |
| [Evals in CI with Promptfoo](guides/evals-in-ci-with-promptfoo.md) | Evaluate | Promptfoo |
| [Pytest-style evals with DeepEval](guides/evals-pytest-style-with-deepeval.md) | Evaluate | DeepEval |
| [Runtime guardrails with NeMo & Guardrails AI](guides/runtime-guardrails-nemo-and-guardrails-ai.md) | Guard | NeMo Guardrails, Guardrails AI |
| [Red-teaming LLMs with garak](guides/red-teaming-with-garak.md) | Guard | garak |

## Tracing & Observability Platforms

End-to-end tracing of LLM calls, tool invocations, and multi-step agent runs.

> 📘 **Hands-on:** [Tracing a Python agent with Langfuse](guides/tracing-python-agent-with-langfuse.md) · [Tracing with Arize Phoenix](guides/tracing-with-arize-phoenix.md) · [Vendor-neutral tracing with OpenTelemetry](guides/vendor-neutral-tracing-with-opentelemetry.md)

### Comparison

Quickly narrow the field. *Self-hosted?* means an official open-source deployment you can run yourself (vs. managed-first). *OTel-native* means traces follow OpenTelemetry conventions and export to other backends.

| Tool | Self-hosted? | OTel-native | Primary SDKs | License | Best for |
|---|---|---|---|---|---|
| [Langfuse](https://github.com/langfuse/langfuse) | ✅ | ✅ | Python, JS/TS | MIT (open-core) | All-in-one: tracing + evals + prompts |
| [Arize Phoenix](https://github.com/Arize-ai/phoenix) | ✅ | ✅ | Python, JS/TS | Elastic v2 | Local-first debugging + built-in evals |
| [OpenLLMetry](https://github.com/traceloop/openllmetry) | ✅ (backend-agnostic) | ✅ | Python, JS/TS, Go, Ruby | Apache-2.0 | Shipping OTel traces to any backend |
| [Opik](https://github.com/comet-ml/opik) | ✅ | ✅ | Python, TS | Apache-2.0 | Open-source tracing + evals |
| [OpenLIT](https://github.com/openlit/openlit) | ✅ | ✅ | Python, TS | Apache-2.0 | One-line OTel instrumentation (incl. GPUs) |
| [Helicone](https://github.com/Helicone/helicone) | ✅ | Proxy / OTel | Any (proxy) | Apache-2.0 | Minimal-code logging via a proxy |
| [Laminar](https://github.com/lmnr-ai/lmnr) | ✅ | ✅ | Python, JS/TS | Apache-2.0 | Tracing + evals on ClickHouse |
| [Langtrace](https://github.com/Scale3-Labs/langtrace) | ✅ | ✅ | Python, TS | AGPL-3.0 | OTel-based, framework-agnostic tracing |
| [AgentOps](https://github.com/AgentOps-AI/agentops) | Cloud | Partial | Python | MIT | Agent session replays & cost tracking |
| [Pydantic Logfire](https://github.com/pydantic/logfire) | Cloud | ✅ | Python | MIT (SDK) | Pydantic / Python-stack apps |
| [LangWatch](https://github.com/langwatch/langwatch) | ✅ | ✅ | Python, TS | Apache-2.0 (open-core) | Monitoring + evals + analytics |
| [Weave](https://github.com/wandb/weave) | Cloud | Partial | Python, TS | Apache-2.0 | Teams already on Weights & Biases |
| [MLflow](https://github.com/mlflow/mlflow) | ✅ | ✅ | Python | Apache-2.0 | Teams already using MLflow |

### All entries

- [Langfuse](https://github.com/langfuse/langfuse) — Open-source LLM engineering platform with tracing, evals, prompt management, and datasets; OpenTelemetry-native.
- [Arize Phoenix](https://github.com/Arize-ai/phoenix) — Open-source observability for tracing, evaluating, and debugging agents, built on OpenTelemetry.
- [OpenLLMetry](https://github.com/traceloop/openllmetry) — OpenTelemetry extensions that add complete observability to LLM apps and ship traces to existing backends.
- [OpenLLMetry JS](https://github.com/traceloop/openllmetry-js) — The JavaScript/TypeScript counterpart of OpenLLMetry for Node and browser agents.
- [Opik](https://github.com/comet-ml/opik) — Debug, evaluate, and monitor LLM apps, RAG, and agentic workflows with tracing and production dashboards.
- [OpenLIT](https://github.com/openlit/openlit) — OpenTelemetry-native observability for LLMs, agents, and GPUs with one-line instrumentation.
- [Helicone](https://github.com/Helicone/helicone) — Proxy-based observability for logging, monitoring, and debugging LLM and agent traffic.
- [Laminar](https://github.com/lmnr-ai/lmnr) — Open-source platform for tracing and evaluating AI agents, built on OpenTelemetry and ClickHouse.
- [Langtrace](https://github.com/Scale3-Labs/langtrace) — Open-source, OpenTelemetry-based tracing for LLM apps and agent frameworks.
- [AgentOps](https://github.com/AgentOps-AI/agentops) — Session replays, cost tracking, and analytics purpose-built for AI agents.
- [Pydantic Logfire](https://github.com/pydantic/logfire) — Observability from the Pydantic team with first-class LLM and agent tracing on OpenTelemetry.
- [LangWatch](https://github.com/langwatch/langwatch) — Monitoring, evaluation, and analytics platform for LLM and agent applications.
- [Weave](https://github.com/wandb/weave) — Weights & Biases toolkit for tracking, evaluating, and debugging LLM application calls.
- [MLflow](https://github.com/mlflow/mlflow) — ML lifecycle platform with LLM tracing, evaluation, and prompt management features.
- [TWZRD Agent Intel](https://intel.twzrd.xyz) — Agent wallet trust scoring for multi-agent observability pipelines. Verify AI agent identity before logging agent actions or granting production access. Free MCP with `score_agent(wallet)` and `preflight_check(wallet)` — zero-install via streamable HTTP.

## Evaluation Frameworks

Score agent trajectories, tool use, and output quality — in CI and in production.

> 📘 **Hands-on:** [Evals in CI with Promptfoo](guides/evals-in-ci-with-promptfoo.md) · [Pytest-style evals with DeepEval](guides/evals-pytest-style-with-deepeval.md)

### Comparison

| Tool | Approach | Primary SDK | License | Best for |
|---|---|---|---|---|
| [Promptfoo](https://github.com/promptfoo/promptfoo) | Config-driven CLI + matrix | Node / CLI (any backend) | MIT | CI quality gates, model/prompt comparison |
| [DeepEval](https://github.com/confident-ai/deepeval) | `pytest`-style unit tests | Python | Apache-2.0 | Evals-as-tests living next to your code |
| [Ragas](https://github.com/vibrantlabsai/ragas) | RAG/agent metrics library | Python | Apache-2.0 | RAG & multi-turn pipeline evaluation |
| [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) | Scripted eval framework | Python | MIT | Rigorous, reproducible eval plans |
| [TruLens](https://github.com/truera/trulens) | Feedback functions + tracing | Python | MIT | Groundedness / relevance feedback |
| [Giskard](https://github.com/Giskard-AI/giskard-oss) | Automated scanning | Python | Apache-2.0 | Auto-scan for hallucination, bias, robustness |
| [Evidently](https://github.com/evidentlyai/evidently) | Metrics + monitoring | Python | Apache-2.0 | ML + LLM monitoring with 100+ metrics |

### All entries

- [DeepEval](https://github.com/confident-ai/deepeval) — Pytest-like framework for unit-testing LLM apps with 30+ built-in metrics.
- [Ragas](https://github.com/vibrantlabsai/ragas) — Evaluation toolkit for RAG and multi-turn, multi-tool agents.
- [Promptfoo](https://github.com/promptfoo/promptfoo) — Local-first CLI and dashboard for evaluating prompts, RAG flows, and agents with regression detection.
- [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) — UK AI Safety Institute framework for scripted eval plans, tool calls, and model-graded rubrics.
- [OpenAI Evals](https://github.com/openai/evals) — Framework and registry for evaluating LLMs and LLM systems.
- [OpenAI simple-evals](https://github.com/openai/simple-evals) — Lightweight reference implementations of common LLM benchmarks.
- [agentevals](https://github.com/langchain-ai/agentevals) — LangChain utilities for evaluating agent trajectories and tool-calling behavior.
- [openevals](https://github.com/langchain-ai/openevals) — Ready-to-use LLM-as-judge and heuristic evaluators from LangChain.
- [TruLens](https://github.com/truera/trulens) — Instrument, evaluate, and track LLM apps with feedback functions for groundedness and relevance.
- [Giskard](https://github.com/Giskard-AI/giskard-oss) — Testing framework that scans LLM and ML systems for hallucinations, bias, and robustness issues.
- [Evidently](https://github.com/evidentlyai/evidently) — Evaluation and monitoring for ML and LLM systems with 100+ metrics and a test suite API.
- [Deepchecks](https://github.com/deepchecks/deepchecks) — Continuous validation and testing for LLM apps and ML models.
- [UpTrain](https://github.com/uptrain-ai/uptrain) — Open-source toolkit to evaluate and improve LLM apps with pre-built and custom checks.
- [continuous-eval](https://github.com/relari-ai/continuous-eval) — Data-driven evaluation for LLM-powered pipelines and modular agent chains.
- [Patronus SDK](https://github.com/patronus-ai/patronus-py) — Python SDK for evaluating and monitoring LLM and agent outputs against managed evaluators.
- [Agent Evaluation](https://github.com/awslabs/agent-evaluation) — AWS framework that uses an LLM judge to test conversational agents end-to-end.
- [Agenta](https://github.com/Agenta-AI/agenta) — Open-source LLMOps platform for prompt engineering, evaluation, and observability.

## Guardrails & Safety

Enforce policies and constrain agent inputs and outputs at runtime.

> 📘 **Hands-on:** [Runtime guardrails with NeMo & Guardrails AI](guides/runtime-guardrails-nemo-and-guardrails-ai.md)

- [NeMo Guardrails](https://github.com/NVIDIA-NeMo/Guardrails) — NVIDIA toolkit for adding programmable guardrails to LLM-based systems via the Colang DSL.
- [Guardrails AI](https://github.com/guardrails-ai/guardrails) — Input/output guards built from composable validators that detect and mitigate specific risks.
- [LLM Guard](https://github.com/protectai/llm-guard) — Security toolkit that sanitizes prompts and responses by chaining multiple scanners.
- [LangKit](https://github.com/whylabs/langkit) — Toolkit for extracting safety, quality, and security signals from prompts and responses.

## Red Teaming & Security Testing

Probe agents for prompt injection, jailbreaks, and other failure modes before attackers do.

> 📘 **Hands-on:** [Red-teaming LLMs with garak](guides/red-teaming-with-garak.md)

- [garak](https://github.com/NVIDIA/garak) — LLM vulnerability scanner that probes for hallucination, jailbreaks, prompt injection, and more.
- [Rebuff](https://github.com/protectai/rebuff) — Self-hardening prompt-injection detector for protecting LLM apps.
- [PyRIT](https://github.com/Azure/PyRIT) — Microsoft's Python Risk Identification Tool for red-teaming generative AI systems.

## Instrumentation & Standards

The shared semantics that let traces flow between tools.

> 📘 **Hands-on:** [Vendor-neutral tracing with OpenTelemetry](guides/vendor-neutral-tracing-with-opentelemetry.md)

- [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — The emerging standard for representing LLM and agent spans in OpenTelemetry.
- [OpenInference](https://github.com/Arize-ai/openinference) — OpenTelemetry-compatible instrumentation conventions and libraries for LLM and agent frameworks.
- [OpenTelemetry Python Contrib](https://github.com/open-telemetry/opentelemetry-python-contrib) — Community instrumentations, including GenAI libraries, for OpenTelemetry Python.

## Benchmarks

Standardized tasks for measuring agent capability and regressions over time.

- [SWE-bench](https://github.com/SWE-bench/SWE-bench) — Benchmark of real GitHub issues for evaluating coding agents on software engineering tasks.
- [WebArena](https://github.com/web-arena-x/webarena) — Self-hostable web environment for benchmarking autonomous web agents on realistic tasks.
- [AgentBench](https://github.com/THUDM/AgentBench) — Multi-environment benchmark for evaluating LLMs as agents across diverse tasks.
- [HELM](https://github.com/stanford-crfm/helm) — Stanford's Holistic Evaluation of Language Models framework and leaderboard.
- [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) — EleutherAI's de facto framework for few-shot evaluation of language models.

## Gateways with Built-in Observability

LLM proxies that emit logs, traces, and cost metrics for everything behind them.

- [LiteLLM](https://github.com/BerriAI/litellm) — Unified gateway to 100+ LLMs with built-in logging, cost tracking, and observability hooks.
- [Portkey Gateway](https://github.com/portkey-ai/gateway) — Fast open-source AI gateway with tracing, guardrails, and observability across providers.

## Related Lists

- [Awesome LLMOps](https://github.com/tensorchord/Awesome-LLMOps) — Broad curated list of LLMOps tooling.
- [Awesome AI Eval](https://github.com/Vvkmnn/awesome-ai-eval) — Curated tools and methods for evaluating AI reliability.
- [Awesome Harness Engineering](https://github.com/ai-boost/awesome-harness-engineering) — Patterns and tools for AI agent harness engineering, including evals and observability.

## Contributing

Contributions are welcome! Please read the [contribution guidelines](CONTRIBUTING.md) first. Every entry must be a real, actively maintained project with a working link.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, Daniel Tsionit has waived all copyright and related or neighboring rights to this work. See [LICENSE](LICENSE).

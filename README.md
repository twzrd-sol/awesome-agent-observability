# Awesome Agent Observability [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of tools for observability, evaluation, tracing, and guardrails of production LLM agents.

Shipping an agent to production is the easy part. Knowing *why* it failed a tool call, regressed after a prompt change, or leaked a secret is the hard part. This list collects the tools, frameworks, and standards that make agentic systems measurable, debuggable, and safe — the "ops" layer that most agent tutorials skip.

Scope: tools focused on **tracing, evaluation, monitoring, and guardrails for LLM agents**. Generic APM and broad LLMOps catalogs are out of scope unless they have first-class agent support.

## Contents

- [Tracing & Observability Platforms](#tracing--observability-platforms)
- [Evaluation Frameworks](#evaluation-frameworks)
- [Guardrails & Safety](#guardrails--safety)
- [Red Teaming & Security Testing](#red-teaming--security-testing)
- [Instrumentation & Standards](#instrumentation--standards)
- [Benchmarks](#benchmarks)
- [Gateways with Built-in Observability](#gateways-with-built-in-observability)
- [Related Lists](#related-lists)

## Tracing & Observability Platforms

End-to-end tracing of LLM calls, tool invocations, and multi-step agent runs.

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

## Evaluation Frameworks

Score agent trajectories, tool use, and output quality — in CI and in production.

- [DeepEval](https://github.com/confident-ai/deepeval) — Pytest-like framework for unit-testing LLM apps with 30+ built-in metrics.
- [Ragas](https://github.com/explodinggradients/ragas) — Evaluation toolkit for RAG and multi-turn, multi-tool agents.
- [Promptfoo](https://github.com/promptfoo/promptfoo) — Local-first CLI and dashboard for evaluating prompts, RAG flows, and agents with regression detection.
- [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) — UK AI Safety Institute framework for scripted eval plans, tool calls, and model-graded rubrics.
- [OpenAI Evals](https://github.com/openai/evals) — Framework and registry for evaluating LLMs and LLM systems.
- [OpenAI simple-evals](https://github.com/openai/simple-evals) — Lightweight reference implementations of common LLM benchmarks.
- [agentevals](https://github.com/langchain-ai/agentevals) — LangChain utilities for evaluating agent trajectories and tool-calling behavior.
- [openevals](https://github.com/langchain-ai/openevals) — Ready-to-use LLM-as-judge and heuristic evaluators from LangChain.
- [TruLens](https://github.com/truera/trulens) — Instrument, evaluate, and track LLM apps with feedback functions for groundedness and relevance.
- [Giskard](https://github.com/Giskard-AI/giskard) — Testing framework that scans LLM and ML systems for hallucinations, bias, and robustness issues.
- [Evidently](https://github.com/evidentlyai/evidently) — Evaluation and monitoring for ML and LLM systems with 100+ metrics and a test suite API.
- [Deepchecks](https://github.com/deepchecks/deepchecks) — Continuous validation and testing for LLM apps and ML models.
- [UpTrain](https://github.com/uptrain-ai/uptrain) — Open-source toolkit to evaluate and improve LLM apps with pre-built and custom checks.
- [continuous-eval](https://github.com/relari-ai/continuous-eval) — Data-driven evaluation for LLM-powered pipelines and modular agent chains.
- [Patronus SDK](https://github.com/patronus-ai/patronus-py) — Python SDK for evaluating and monitoring LLM and agent outputs against managed evaluators.
- [Agent Evaluation](https://github.com/awslabs/agent-evaluation) — AWS framework that uses an LLM judge to test conversational agents end-to-end.
- [Agenta](https://github.com/Agenta-AI/agenta) — Open-source LLMOps platform for prompt engineering, evaluation, and observability.

## Guardrails & Safety

Enforce policies and constrain agent inputs and outputs at runtime.

- [NeMo Guardrails](https://github.com/NVIDIA-NeMo/Guardrails) — NVIDIA toolkit for adding programmable guardrails to LLM-based systems via the Colang DSL.
- [Guardrails AI](https://github.com/guardrails-ai/guardrails) — Input/output guards built from composable validators that detect and mitigate specific risks.
- [LLM Guard](https://github.com/protectai/llm-guard) — Security toolkit that sanitizes prompts and responses by chaining multiple scanners.
- [LangKit](https://github.com/whylabs/langkit) — Toolkit for extracting safety, quality, and security signals from prompts and responses.

## Red Teaming & Security Testing

Probe agents for prompt injection, jailbreaks, and other failure modes before attackers do.

- [garak](https://github.com/leondz/garak) — LLM vulnerability scanner that probes for hallucination, jailbreaks, prompt injection, and more.
- [Rebuff](https://github.com/protectai/rebuff) — Self-hardening prompt-injection detector for protecting LLM apps.
- [PyRIT](https://github.com/Azure/PyRIT) — Microsoft's Python Risk Identification Tool for red-teaming generative AI systems.

## Instrumentation & Standards

The shared semantics that let traces flow between tools.

- [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — The emerging standard for representing LLM and agent spans in OpenTelemetry.
- [OpenInference](https://github.com/Arize-ai/openinference) — OpenTelemetry-compatible instrumentation conventions and libraries for LLM and agent frameworks.
- [OpenTelemetry Python Contrib](https://github.com/open-telemetry/opentelemetry-python-contrib) — Community instrumentations, including GenAI libraries, for OpenTelemetry Python.

## Benchmarks

Standardized tasks for measuring agent capability and regressions over time.

- [SWE-bench](https://github.com/princeton-nlp/SWE-bench) — Benchmark of real GitHub issues for evaluating coding agents on software engineering tasks.
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

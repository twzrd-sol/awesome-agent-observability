# Tracing with Arize Phoenix

[Arize Phoenix](https://github.com/Arize-ai/phoenix) is open-source, OpenTelemetry-based observability you can run **locally with one command**. It uses [OpenInference](https://github.com/Arize-ai/openinference) auto-instrumentors, so you rarely write tracing code by hand — you install an instrumentor for your framework and traces appear. Great when you want a self-hosted UI with zero cloud signup.

> **Time:** ~10 minutes · **Level:** beginner

## What you'll build

A local Phoenix server plus an OpenAI app whose every call is captured and visualized as spans — input messages, output, model, tokens, and latency — viewable at `http://localhost:6006`.

## Prerequisites

- Python 3.9+
- An OpenAI API key

## 1. Install

```bash
pip install arize-phoenix openinference-instrumentation-openai openai
```

- `arize-phoenix` — the server + UI and the `phoenix.otel` helpers.
- `openinference-instrumentation-openai` — auto-instruments the OpenAI SDK.

## 2. Start the Phoenix server

In a **separate terminal**, launch the local collector + UI:

```bash
phoenix serve
```

It boots at `http://localhost:6006`. Leave it running. (Prefer Docker? `docker run -p 6006:6006 -p 4317:4317 arizephoenix/phoenix:latest`.)

## 3. Set credentials

```bash
export OPENAI_API_KEY="sk-..."
export PHOENIX_COLLECTOR_ENDPOINT="http://localhost:6006"
```

For Phoenix Cloud instead of local, also set `PHOENIX_API_KEY` and point `PHOENIX_COLLECTOR_ENDPOINT` at your cloud URL.

## 4. Minimal working example

`register()` wires up the OpenTelemetry tracer provider and points it at your Phoenix server. `auto_instrument=True` activates every OpenInference instrumentor you have installed.

```python
# app.py
from phoenix.otel import register
from openai import OpenAI

# Configure the tracer provider and auto-instrument installed libraries.
# PHOENIX_COLLECTOR_ENDPOINT is read from the environment.
tracer_provider = register(
    project_name="support-agent",
    auto_instrument=True,
)

client = OpenAI()

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Explain OpenTelemetry spans in one sentence."}],
)
print(resp.choices[0].message.content)
```

Run it (with `phoenix serve` still running in the other terminal):

```bash
python app.py
```

> Prefer explicit control over `auto_instrument`? Drop that flag and instrument manually:
> ```python
> from openinference.instrumentation.openai import OpenAIInstrumentor
> OpenAIInstrumentor().instrument(tracer_provider=tracer_provider)
> ```

## 5. What you'll see

Open `http://localhost:6006`, select the **support-agent** project, and open the latest trace. Each LLM call is a span showing:

- The full request messages and the model's response.
- Model name, prompt/completion **token counts**, and **latency**.
- The OpenInference span kind (`LLM`, `CHAIN`, `TOOL`, `AGENT`), which lets you filter the trace tree.

You can also run Phoenix's built-in LLM **evals** (hallucination, relevance, toxicity) over captured traces directly from the UI — see the eval docs below.

## 6. Tracing a framework agent

Phoenix has OpenInference instrumentors for most frameworks. Install the matching package and `auto_instrument=True` picks it up:

```bash
pip install openinference-instrumentation-langchain      # LangChain / LangGraph
pip install openinference-instrumentation-llama-index    # LlamaIndex
pip install openinference-instrumentation-openai-agents  # OpenAI Agents SDK
```

No other code changes — `register(auto_instrument=True)` detects installed instrumentors at startup.

## Common pitfalls

- **No traces appear.** The server isn't running, or `PHOENIX_COLLECTOR_ENDPOINT` doesn't match where `phoenix serve` is listening (`http://localhost:6006`). Start Phoenix *before* running your app.
- **Traces vanish on restart.** Default `phoenix serve` keeps data in memory. For persistence set `PHOENIX_WORKING_DIR` (SQLite) or use the Postgres-backed Docker deployment.
- **Instrumentor not active.** `auto_instrument=True` only activates instrumentors that are actually `pip install`ed. Missing spans usually means the instrumentation package for that library isn't installed.
- **Port already in use.** Something else is on `6006`. Stop it or run `phoenix serve --port 6007` and update `PHOENIX_COLLECTOR_ENDPOINT`.

## Official docs

- Quickstart (tracing): https://arize.com/docs/phoenix/tracing/llm-traces-1
- Self-hosting: https://arize.com/docs/phoenix/self-hosting
- OpenInference instrumentors: https://github.com/Arize-ai/openinference
- Running evals over traces: https://arize.com/docs/phoenix/evaluation/llm-evals

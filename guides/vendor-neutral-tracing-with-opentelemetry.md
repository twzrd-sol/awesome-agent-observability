# Vendor-Neutral Tracing with OpenTelemetry

If you don't want to commit to one observability vendor, instrument your agent with **plain OpenTelemetry** and the emerging [GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/). Your traces then flow to *any* OTLP-compatible backend — Jaeger, Grafana Tempo, Langfuse, Phoenix, Honeycomb, Datadog — without rewriting instrumentation. This guide uses [OpenLLMetry](https://github.com/traceloop/openllmetry) (Traceloop's SDK) as the thin, OTel-standard instrumentation layer.

> **Time:** ~15 minutes · **Level:** intermediate

## Why this approach

- **No lock-in.** Spans follow the OTel GenAI conventions (`gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.*`). Swap backends by changing one env var.
- **One pipeline for everything.** Your LLM spans sit in the same trace as your HTTP, DB, and queue spans.
- **Standards-based.** OpenLLMetry emits OpenTelemetry spans; it doesn't invent a proprietary format.

## Prerequisites

- Python 3.9+
- An OpenAI API key
- Docker (to run a local Jaeger backend for this demo)

## 1. Run a local OTLP backend (Jaeger)

Jaeger's all-in-one image accepts OTLP and gives you a trace UI:

```bash
docker run --rm --name jaeger \
  -p 16686:16686 \
  -p 4318:4318 \
  jaegertracing/all-in-one:latest
```

- UI: `http://localhost:16686`
- OTLP/HTTP receiver: `http://localhost:4318`

## 2. Install

```bash
pip install traceloop-sdk openai
```

`traceloop-sdk` bundles the OpenTelemetry SDK plus auto-instrumentors for OpenAI, Anthropic, LangChain, LlamaIndex, and more.

## 3. Set credentials and the OTLP target

```bash
export OPENAI_API_KEY="sk-..."
export OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4318"
# If your backend needs auth (e.g. a SaaS), add headers:
# export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer <token>"
```

Because you set the standard `OTEL_EXPORTER_OTLP_ENDPOINT`, OpenLLMetry ships spans there instead of to Traceloop's cloud.

## 4. Minimal working example

```python
# app.py
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow, task
from openai import OpenAI

# Reads OTEL_EXPORTER_OTLP_ENDPOINT from the env and auto-instruments OpenAI.
# disable_batch=True flushes immediately — convenient for local/dev scripts.
Traceloop.init(app_name="research-agent", disable_batch=True)

client = OpenAI()


@task(name="draft_search_query")
def draft_query(topic: str) -> str:
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": f"Write one web search query for: {topic}"}],
    )
    return resp.choices[0].message.content


@workflow(name="research")
def research(topic: str) -> str:
    query = draft_query(topic)
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": f"Answer the query: {query}"}],
    )
    return resp.choices[0].message.content


if __name__ == "__main__":
    print(research("vector databases for RAG"))
```

Run it (Jaeger still running):

```bash
python app.py
```

## 5. What you'll see

Open Jaeger at `http://localhost:16686`, pick the **research-agent** service, and click **Find Traces**. You'll see one trace per `research()` run with:

- A `research` workflow span wrapping a `draft_search_query` task span.
- Two OpenAI generation spans carrying GenAI-convention attributes: `gen_ai.system=openai`, `gen_ai.request.model=gpt-4o-mini`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, and the prompt/response content.

Because these are standard OTel spans, the *same* code exports to any other backend — just change `OTEL_EXPORTER_OTLP_ENDPOINT`.

## 6. Want to see spans without any backend?

For a quick sanity check, print spans to your console:

```python
from opentelemetry.sdk.trace.export import ConsoleSpanExporter
from traceloop.sdk import Traceloop

Traceloop.init(app_name="research-agent", exporter=ConsoleSpanExporter(), disable_batch=True)
```

## 7. Switching backends (the whole point)

Same instrumentation, different destinations — just env vars:

| Backend | `OTEL_EXPORTER_OTLP_ENDPOINT` | Notes |
|---|---|---|
| Jaeger (local) | `http://localhost:4318` | this guide |
| Grafana Tempo | your Tempo OTLP endpoint | add tenant headers if needed |
| Langfuse | `https://cloud.langfuse.com/api/public/otel` | Basic-auth header from LF keys |
| Honeycomb / Datadog / etc. | vendor OTLP URL | set `OTEL_EXPORTER_OTLP_HEADERS` for the API key |

## Common pitfalls

- **Nothing in the UI.** The exporter endpoint is wrong or the backend isn't up. Confirm Jaeger's OTLP port is `4318` (HTTP). For gRPC exporters the port is `4317` — match the protocol your SDK uses.
- **Spans never flush in short scripts.** Use `disable_batch=True` in dev, or call the SDK's flush/shutdown before exit in production batch jobs.
- **No prompt/response content.** Some setups disable content capture for privacy. Check `TRACELOOP_TRACE_CONTENT` (defaults to enabled) if message bodies are missing.
- **GenAI attributes still evolving.** The semantic conventions are not yet stable (1.0); attribute names can change between OTel releases. Pin versions and re-check the spec when upgrading.

## Official docs

- OTel GenAI semantic conventions: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- OpenLLMetry SDK: https://www.traceloop.com/docs/openllmetry/introduction
- OpenTelemetry Python: https://opentelemetry.io/docs/languages/python/
- Jaeger getting started: https://www.jaegertracing.io/docs/latest/getting-started/

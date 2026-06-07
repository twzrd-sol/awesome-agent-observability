# Tracing a Python Agent with Langfuse

[Langfuse](https://github.com/langfuse/langfuse) is an open-source LLM engineering platform: tracing, evals, prompt management, and datasets. It is OpenTelemetry-native, so the same traces can later be shipped elsewhere. This guide gets you from zero to a fully traced agent run you can inspect in the Langfuse UI.

> **Time:** ~10 minutes · **Level:** beginner

## What you'll build

A small OpenAI-backed function, traced end-to-end. You'll see the input, output, model, token counts, latency, and cost for every LLM call — plus the nesting between your own functions and the model calls inside them.

## Prerequisites

- Python 3.9+
- An OpenAI API key (or any provider Langfuse's OpenAI wrapper supports)
- A Langfuse project. Either:
  - **Cloud (fastest):** sign up at https://cloud.langfuse.com and create a project, or
  - **Self-host (free, your data):** see [Self-hosting](#self-hosting) below.

## 1. Install

```bash
pip install langfuse openai
```

## 2. Set credentials

Langfuse reads these from the environment. Grab the keys from **Project Settings → API Keys**.

```bash
export LANGFUSE_PUBLIC_KEY="pk-lf-..."
export LANGFUSE_SECRET_KEY="sk-lf-..."
export LANGFUSE_HOST="https://cloud.langfuse.com"   # or http://localhost:3000 if self-hosting
export OPENAI_API_KEY="sk-..."
```

> Never commit these. Use a `.env` file (and `.gitignore` it) or your secrets manager.

## 3. Minimal working example

The simplest path is the drop-in OpenAI wrapper — import `openai` from `langfuse.openai` and every call is traced automatically. Add the `@observe` decorator to capture your own functions as parent spans.

```python
# agent.py
from langfuse import observe, get_client
from langfuse.openai import openai  # drop-in replacement; auto-traces every call


@observe()  # creates a span for this function
def look_up_policy(question: str) -> str:
    # pretend this hits a vector store / tool
    return "Refunds are accepted within 30 days with a receipt."


@observe()  # parent span; the OpenAI call below nests inside it
def answer(question: str) -> str:
    context = look_up_policy(question)
    response = openai.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": f"Answer using only this policy: {context}"},
            {"role": "user", "content": question},
        ],
    )
    return response.choices[0].message.content


if __name__ == "__main__":
    print(answer("Can I return a blender I bought three weeks ago?"))

    # Traces are sent in the background. Flush before the process exits
    # (scripts/serverless), otherwise you may lose the last batch.
    get_client().flush()
```

Run it:

```bash
python agent.py
```

## 4. What you'll see

Open your Langfuse project → **Tracing → Traces**. You'll find one trace per `answer()` call containing:

- A root span `answer` with the full question and final answer.
- A nested span `look_up_policy` (your tool).
- A nested **generation** for the OpenAI call, showing the exact messages, the model (`gpt-4o-mini`), prompt/completion **token counts**, **latency**, and **estimated cost**.

Click the generation to see the rendered prompt and response side by side — this is where you debug "why did it say that?".

## 5. Tracing a framework agent (LangChain)

If your agent uses LangChain/LangGraph, use the callback handler instead of the OpenAI wrapper:

```bash
pip install langfuse langchain langchain-openai
```

```python
from langfuse.langchain import CallbackHandler
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

handler = CallbackHandler()  # reads LANGFUSE_* env vars

chain = ChatPromptTemplate.from_template("Summarize: {text}") | ChatOpenAI(model="gpt-4o-mini")
chain.invoke({"text": "..."}, config={"callbacks": [handler]})
```

Every chain step, tool call, and LLM call shows up as a nested span automatically.

## 6. Add scores and metadata (optional but powerful)

Attach evaluation scores or user/session IDs to a trace so you can slice metrics later:

```python
from langfuse import observe, get_client

@observe()
def answer(question: str) -> str:
    langfuse = get_client()
    langfuse.update_current_trace(user_id="user_42", session_id="sess_abc",
                                  tags=["prod", "support-bot"])
    ...
    langfuse.score_current_trace(name="helpfulness", value=0.9)
    ...
```

## Self-hosting

Run the full stack locally with Docker:

```bash
git clone https://github.com/langfuse/langfuse
cd langfuse
docker compose up   # web UI on http://localhost:3000
```

Create a user + project in the UI, copy the API keys, and set `LANGFUSE_HOST=http://localhost:3000`. The compose file bundles Postgres, ClickHouse, Redis, and MinIO — fine for evaluation; review the [self-hosting docs](https://langfuse.com/self-hosting) before production.

## Common pitfalls

- **Empty dashboard.** You forgot `get_client().flush()` in a short-lived script, or the process exited before the background exporter sent the batch. Always flush at the end of scripts and serverless handlers.
- **Wrong host.** `LANGFUSE_HOST` must point at the *server* (`http://localhost:3000` for self-host, `https://cloud.langfuse.com` for EU cloud, `https://us.cloud.langfuse.com` for US cloud). A mismatch fails silently.
- **Auth errors.** Public key starts with `pk-lf-`, secret with `sk-lf-`. They must belong to the same project.
- **Costs show as zero.** Langfuse infers cost from the model name. Custom/proxied model names may need a [model definition](https://langfuse.com/docs/model-usage-and-cost) to map to a price.
- **Using the plain `import openai`.** Only `from langfuse.openai import openai` is auto-traced. The standard import is not.

## Official docs

- Python SDK: https://langfuse.com/docs/sdk/python
- OpenAI integration: https://langfuse.com/docs/integrations/openai/python
- LangChain integration: https://langfuse.com/docs/integrations/langchain
- Self-hosting: https://langfuse.com/self-hosting

# Evals in CI with Promptfoo

[Promptfoo](https://github.com/promptfoo/promptfoo) is a local-first, config-driven eval runner. You declare prompts, providers, and test cases with assertions in a YAML file, run `promptfoo eval`, and get a pass/fail matrix. It needs no SDK in your app and drops cleanly into CI as a **quality gate** that blocks regressions before they merge.

> **Time:** ~15 minutes · **Level:** beginner–intermediate

## What you'll build

A `promptfooconfig.yaml` that tests an LLM prompt against several inputs with automatic assertions, runs locally, and then runs in GitHub Actions to fail the build when output quality drops.

## Prerequisites

- Node.js 18+ (Promptfoo runs via `npx`, no install needed)
- An OpenAI API key (or any [supported provider](https://www.promptfoo.dev/docs/providers/))

## 1. Initialize a project

```bash
npx promptfoo@latest init
```

This scaffolds a `promptfooconfig.yaml`. Set your key:

```bash
export OPENAI_API_KEY="sk-..."
```

## 2. Write the config

Replace the generated file with a minimal, real example. It tests a support-classifier prompt across three inputs, mixing deterministic and model-graded assertions.

```yaml
# promptfooconfig.yaml
description: "Support ticket classifier"

prompts:
  - |
    Classify the support ticket below into exactly one of:
    BILLING, TECHNICAL, ACCOUNT, OTHER.
    Reply with only the label.

    Ticket: {{ticket}}

providers:
  - openai:gpt-4o-mini

tests:
  - vars:
      ticket: "My card was charged twice this month."
    assert:
      - type: contains
        value: "BILLING"

  - vars:
      ticket: "The app crashes when I upload a PDF."
    assert:
      - type: contains
        value: "TECHNICAL"

  - vars:
      ticket: "How do I change my email address?"
    assert:
      # model-graded check for free-form correctness
      - type: llm-rubric
        value: "The label is ACCOUNT and no other text is included."
```

Useful assertion types: `contains`, `equals`, `regex`, `is-json`, `cost`, `latency` (deterministic) and `llm-rubric`, `similar`, `factuality` (model-graded). Full list: the [assertions docs](https://www.promptfoo.dev/docs/configuration/expected-outputs/).

## 3. Run locally

```bash
npx promptfoo@latest eval
```

Then open the interactive results table:

```bash
npx promptfoo@latest view
```

## 4. What you'll see

The terminal prints a matrix: one row per test case, one column per provider, each cell PASS/FAIL with the model's actual output and which assertion failed. The web view (`promptfoo view`) adds a sortable grid, per-cell diffs, token cost, and latency — ideal for eyeballing *why* a case failed and comparing two models side by side.

The process exits **non-zero if any test fails** — that's what makes it a CI gate.

## 5. Add it to CI (GitHub Actions)

Create `.github/workflows/llm-eval.yml`. The eval runs on every PR; if any assertion fails, the job fails and blocks merge.

```yaml
name: LLM Eval
on:
  pull_request:
    paths:
      - "promptfooconfig.yaml"
      - "prompts/**"
      - ".github/workflows/llm-eval.yml"

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Run promptfoo eval
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: npx promptfoo@latest eval --output results.json

      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: eval-results
          path: results.json
```

Add `OPENAI_API_KEY` under **Repo → Settings → Secrets and variables → Actions**. Because `promptfoo eval` exits non-zero on failure, the `Run promptfoo eval` step fails the job automatically — no extra gate script needed. (Want a softer gate? Use the official [`promptfoo-action`](https://github.com/promptfoo/promptfoo-action), which can comment results on the PR.)

## 6. Testing a real agent endpoint

Instead of a raw prompt, point a provider at your own HTTP endpoint or a Python script:

```yaml
providers:
  - id: "https://your-agent.example.com/chat"
    config:
      method: POST
      headers: { "Content-Type": "application/json" }
      body: { "message": "{{ticket}}" }
      transformResponse: "json.reply"
  # or run local code:
  - "exec:python agent_provider.py"
```

This lets you eval the *whole agent* (tools, retrieval, post-processing), not just the model.

## Common pitfalls

- **Flaky `llm-rubric` results.** Model-graded assertions are themselves LLM calls and can be non-deterministic. Set a low temperature on the grader, write tight rubric criteria, or prefer deterministic assertions where possible.
- **CI cost/rate limits.** Each test case is a real API call. Scope the workflow `paths:` so evals only run when prompts change, and keep the suite focused.
- **Caching surprises.** Promptfoo caches results by default; if you expect fresh calls, pass `--no-cache`.
- **Secret not set.** A missing `OPENAI_API_KEY` in CI shows up as provider auth errors, not assertion failures — check the job log's first lines.

## Official docs

- Getting started: https://www.promptfoo.dev/docs/getting-started/
- Configuration reference: https://www.promptfoo.dev/docs/configuration/guide/
- Assertions & metrics: https://www.promptfoo.dev/docs/configuration/expected-outputs/
- CI/CD integration: https://www.promptfoo.dev/docs/integrations/ci-cd/

# Pytest-Style Evals with DeepEval

[DeepEval](https://github.com/confident-ai/deepeval) lets you unit-test LLM output the way you already test code: write test functions, assert on metrics, run them with a `pytest`-like runner. It ships 30+ research-backed metrics (relevancy, faithfulness, hallucination, plus customizable `GEval` rubrics). Use it when you want evals living next to your code, gated by CI, and expressed as familiar test assertions.

> **Time:** ~15 minutes · **Level:** intermediate

## What you'll build

A test file with two LLM eval cases — one using a built-in metric, one using a custom `GEval` rubric — runnable with `deepeval test run`, ready to wire into CI.

## Prerequisites

- Python 3.9+
- An OpenAI API key (DeepEval's default metrics use an LLM as judge)

## 1. Install

```bash
pip install -U deepeval
```

## 2. Set credentials

Most metrics are **LLM-as-judge**, so they need a model key:

```bash
export OPENAI_API_KEY="sk-..."
```

> You can swap the judge for Azure, local Ollama, or any custom model — see DeepEval's "Using a custom LLM" docs.

## 3. Minimal working example

The core objects: `LLMTestCase` (the thing under test) and a metric you `assert_test` against. `actual_output` is whatever *your* app produced for the input.

```python
# test_agent.py
from deepeval import assert_test
from deepeval.test_case import LLMTestCase, LLMTestCaseParams
from deepeval.metrics import AnswerRelevancyMetric, GEval


def test_answer_relevancy():
    # Replace actual_output with a real call into your app/agent.
    test_case = LLMTestCase(
        input="What is the capital of France?",
        actual_output="The capital of France is Paris.",
    )
    relevancy = AnswerRelevancyMetric(threshold=0.7)
    assert_test(test_case, [relevancy])


def test_correctness_with_rubric():
    correctness = GEval(
        name="Correctness",
        criteria="Determine whether the actual output is factually correct "
                 "given the expected output.",
        evaluation_params=[
            LLMTestCaseParams.ACTUAL_OUTPUT,
            LLMTestCaseParams.EXPECTED_OUTPUT,
        ],
        threshold=0.5,
    )
    test_case = LLMTestCase(
        input="I have a persistent cough and a fever. Should I worry?",
        actual_output="A cough with fever can be a viral infection; "
                      "see a doctor if it worsens or lasts more than a few days.",
        expected_output="A persistent cough and fever may indicate anything from "
                        "a mild viral infection to pneumonia or COVID-19; seek care "
                        "if symptoms worsen, persist, or include breathing trouble.",
    )
    assert_test(test_case, [correctness])
```

## 4. Run it

Use DeepEval's runner (a thin wrapper over pytest that adds reporting):

```bash
deepeval test run test_agent.py
```

Plain `pytest test_agent.py` also works, but `deepeval test run` gives you the richer summary table and optional cloud reporting.

## 5. What you'll see

A per-test pass/fail report. For each metric you get a **score** (0–1), the **threshold**, pass/fail, and — importantly — a **reason** string explaining the judge's verdict (e.g. *"The output omits the breathing-difficulty warning present in the expected output"*). That reason is the debugging gold: it tells you *why* a case failed, not just that it did.

## 6. Bulk evals with a dataset

For many cases, parametrize over a dataset instead of writing one function each:

```python
import pytest
from deepeval import assert_test
from deepeval.dataset import EvaluationDataset
from deepeval.test_case import LLMTestCase
from deepeval.metrics import AnswerRelevancyMetric

dataset = EvaluationDataset(test_cases=[
    LLMTestCase(input="...", actual_output="..."),
    LLMTestCase(input="...", actual_output="..."),
])

@pytest.mark.parametrize("test_case", dataset.test_cases)
def test_dataset(test_case: LLMTestCase):
    assert_test(test_case, [AnswerRelevancyMetric(threshold=0.7)])
```

## 7. Add it to CI

Since it's pytest-compatible, CI is trivial. In GitHub Actions:

```yaml
- name: Run LLM evals
  env:
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
  run: |
    pip install -U deepeval
    deepeval test run test_agent.py
```

A failing metric returns a non-zero exit code and fails the job.

## Common pitfalls

- **Judge non-determinism.** Metrics are LLM calls, so scores vary run to run. Set sensible `threshold`s with margin, keep rubric `criteria` specific, and avoid borderline thresholds like 0.5 for critical gates.
- **RAG metrics need context.** `FaithfulnessMetric` and `ContextualRelevancyMetric` require `retrieval_context` on the `LLMTestCase`. Omitting it raises an error or produces meaningless scores.
- **Cost.** Every metric on every case is one or more API calls. A 100-case suite with 3 metrics is 300+ calls — scope CI runs and consider a cheaper judge model.
- **`actual_output` must be your app's real output.** Hardcoding the "right" answer tests nothing. Call your agent inside the test (or precompute outputs into a dataset).
- **First-run prompts.** DeepEval may prompt to log in to Confident AI. It's optional — you can run fully locally and skip/ignore the login.

## Official docs

- Getting started: https://deepeval.com/docs/getting-started
- Metrics reference: https://deepeval.com/docs/metrics-introduction
- GEval (custom rubrics): https://deepeval.com/docs/metrics-llm-evals
- Datasets: https://deepeval.com/docs/evaluation-datasets

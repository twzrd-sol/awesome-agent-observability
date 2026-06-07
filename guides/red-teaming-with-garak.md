# Red-Teaming LLMs with garak

[garak](https://github.com/NVIDIA/garak) (now maintained by NVIDIA) is the `nmap` of LLM security: a vulnerability scanner that fires hundreds of known attack probes — prompt injection, jailbreaks, toxicity, data leakage, encoding tricks — at a target model and reports which ones succeeded. Run it *before* attackers do, and re-run it in CI after prompt or model changes.

> **Time:** ~15 minutes · **Level:** intermediate

## What you'll build

A security scan of an LLM target that produces a JSONL report and an HTML summary showing each probe's pass/fail rate — your model's attack surface, quantified.

## Prerequisites

- Python 3.10+ (garak recommends a fresh virtualenv)
- An API key for the target you're scanning (e.g. `OPENAI_API_KEY`), **or** a local model via Hugging Face / Ollama
- Authorization to test the target. Only scan models and endpoints you own or are permitted to test.

## 1. Install

```bash
python -m pip install -U garak
```

Verify:

```bash
garak --version
```

## 2. See what's available

garak ships a large catalog of probes (attack families) and detectors:

```bash
garak --list_probes      # every attack module
garak --list_detectors   # how successes are judged
```

## 3. Minimal working scan

Set your key and run a focused probe. `--target_type` selects the backend, `--target_name` the specific model, `--probes` the attack family.

```bash
export OPENAI_API_KEY="sk-..."

# Scan for encoding-based prompt injection
garak --target_type openai --target_name gpt-4o-mini --probes encoding
```

Other useful probe families:

```bash
# Prompt-injection (PromptInject framework)
garak --target_type openai --target_name gpt-4o-mini --probes promptinject

# DAN / jailbreak attempts
garak --target_type openai --target_name gpt-4o-mini --probes dan
```

> Running every probe (omit `--probes`) is thorough but slow and costly — it's thousands of model calls. Start narrow.

## 4. Scan a local / open model (no API cost)

```bash
# Hugging Face model run locally
garak --target_type huggingface --target_name gpt2 --probes encoding

# Ollama (point at a locally served model)
garak --target_type ollama --target_name llama3 --probes promptinject
```

## 5. What you'll see

garak streams progress per probe, then writes outputs (by default under `~/.local/share/garak/garak_runs/`):

- **`*.report.jsonl`** — every probe attempt and outcome.
- **`*.report.html`** — a human-readable summary with a pass-rate per probe.
- **`*.hitlog.jsonl`** — only the *successful attacks* (the prompts that broke your model). This is the file you act on.

Each probe reports a score like `dan.DAN: 8/10 passed` — meaning 2 attacks succeeded. Lower failure counts after you add guardrails = measurable hardening.

Regenerate/aggregate the HTML report from a JSONL run:

```bash
python -m garak.analyze.report_digest -r path/to/run.report.jsonl -o report.html
```

## 6. Close the loop with guardrails

garak finds holes; guardrails plug them. A practical workflow:

1. Scan baseline → note which probes succeed (check the hitlog).
2. Add input/output guardrails (see [Runtime Guardrails](runtime-guardrails-nemo-and-guardrails-ai.md)).
3. Re-scan → confirm the success counts dropped.
4. Add the scan to CI so regressions resurface automatically.

## Common pitfalls

- **Cost and time blow-ups.** A full scan against a paid API is thousands of calls. Always start with one `--probes` family; use `--generations N` to cap attempts per prompt.
- **Scanning without permission.** garak is an offensive tool. Only point it at models/endpoints you're authorized to test — same rules as any pentest.
- **Flag names changed.** Current garak uses `--target_type` / `--target_name`; older docs and blog posts use `--model_type` / `--model_name`. If a tutorial's flags error out, switch to the `--target_*` form.
- **Detectors aren't perfect.** Some "hits" are false positives (and vice versa). Read the hitlog prompts/responses before declaring a model vulnerable or safe.
- **Non-determinism.** Model randomness means success counts wobble between runs. Compare trends across runs, not single numbers.

## Official docs

- README & quickstart: https://github.com/NVIDIA/garak
- Usage & CLI reference: https://docs.garak.ai
- Probe catalog: https://docs.garak.ai/garak/probes
- Reporting: https://reference.garak.ai/en/latest/reporting.html

<div align="center">

# Maaz Rehman

### I build AI systems that never phone home.

**Four AI systems. Zero API keys. Zero cloud calls. Zero cost per inference.**

<br>

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000000?style=flat-square&logo=ollama&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![ChromaDB](https://img.shields.io/badge/ChromaDB-FF6B35?style=flat-square&logo=databricks&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white)
![Optuna](https://img.shields.io/badge/Optuna-1B5E8C?style=flat-square&logo=optuna&logoColor=white)

</div>

---

## The through-line

Every project on this profile runs **entirely on the machine it's installed on**. The LLM, the
embeddings, the vision model, the vector store — all local, served by [Ollama](https://ollama.com).

That constraint isn't a limitation I worked around. It's the design goal:

> **Nothing you paste ever leaves your laptop, and it costs nothing to run.**

It also forces the interesting engineering. When you can't hide behind a fast hosted endpoint, you
have to actually profile the pipeline, pick the right model size for the job, and prove your
accuracy didn't fall off a cliff when you did.

---

## What I've built

| | Project | The one-line pitch |
|---|---|---|
| 🚨 | **[Triagent](https://github.com/maazkhan211/triagent)** | Paste a 3am stack trace → severity, root cause, and how your team fixed it last time |
| 🔬 | **[AutoML Benchmark Tool](https://github.com/maazkhan211/automl-benchmark-tool)** | AutoML built from scratch — Optuna tuning, leakage-safe pipelines, SHAP explanations |
| 📄 | **[Multimodal RAG](https://github.com/maazkhan211/Multimodal-RAG)** | Treats PDF prose, tables, and figures as three *different* kinds of evidence |
| 💬 | **[Agentic RAG](https://github.com/maazkhan211/Agentic-RAG)** | The foundation — FastAPI + ChromaDB + local embeddings, end to end |

<br>

### 🚨 [Triagent](https://github.com/maazkhan211/triagent) — local-first log triage

Anyone can ask an LLM *"what does this error mean."* The value here is **matching the incident
against what your team already solved** — pulling back the resolution notes from similar past
incidents by vector similarity, so the answer is "we've seen this; here's the fix."

**Why it's more than a wrapper:**

- **3.5× faster, measured.** 173s → 50s per report across three changes — merging two LLM calls
  into one, killing a duplicate embedding, and dropping from an 8B model to a 3B. Each step is
  benchmarked in the README, and the eval score **didn't move**.
- **It ships its own adversarial eval.** Eight logs written specifically to fool a keyword
  classifier. Rule-based baseline: **0/8**. LLM: **5/8**. The three failures are documented
  case-by-case — because a triage aid that hides its error modes isn't a triage aid.
- **Rules *and* LLM, with the disagreement made visible.** A `FATAL` on a staging box isn't an
  outage. The keyword rule can't know that; the LLM can. When they disagree the LLM overrides —
  and you see both verdicts, so the override is never silent.
- **Built for when no human is watching.** The `/webhook` endpoint returns `202 Accepted`
  immediately rather than blocking for ~50s, because alerting tools time out and *retry* — which
  would silently duplicate every triage. It sniffs Sentry, Alertmanager, and Datadog payload
  shapes so it doesn't care which tool fired it.

<br>

### 🔬 [AutoML Benchmark Tool](https://github.com/maazkhan211/automl-benchmark-tool)

**A self-built AutoML tool — not a wrapper around someone else's AutoML library.** Hand it a CSV
and a target column; it detects classification vs. regression, runs a 5-fold cross-validated
Optuna study over each candidate model's search space, benchmarks the winners on an untouched test
set, and explains the winning model with SHAP.

The point is demonstrating the *actual mechanics* of model selection:

- **Leakage-safe by construction** — imputation, encoding, and scaling all live inside an sklearn
  `Pipeline` fit only on training folds.
- **Honest evaluation** — best hyperparameters get refit on full train, then scored once on a
  test set the search never touched.
- **Explanations a human can read** — SHAP feature names are cleaned up (`bp`, not `num__bp`), so
  the charts make sense to someone who doesn't know the preprocessing internals.
- Ships as a Gradio app you can demo live, and exports the winning pipeline as a `.joblib`.

<br>

### 📄 [Multimodal RAG](https://github.com/maazkhan211/Multimodal-RAG)

Most RAG pipelines flatten a PDF into text and lose the two things that carried the actual
information. This one keeps them apart:

- **Tables** → extracted to structured Markdown and retained **verbatim**, so exact lookups stay exact.
- **Figures** → saved as image files and captioned locally by `moondream` for semantic retrieval,
  while the **original image is still shown in the UI as evidence**.
- **Prose** → chunked independently.

Plus a standalone crawler that respects `robots.txt`, rate-limits itself, and verifies
`Content-Type` before downloading a PDF — then folds scraped sites and uploaded documents into a
single merged local knowledge base. Every retrieved item keeps its page number or URL, so the UI
can cite it.

<br>

### 💬 [Agentic RAG](https://github.com/maazkhan211/Agentic-RAG)

The one that started it — a complete local RAG stack with a FastAPI backend, ChromaDB vector
store, sentence-transformers embeddings, Ollama generation, and a plain web chat UI. No hosted
anything.

---

## How I build

**I measure before I optimize.** "Feels slow" isn't a finding. The Triagent README has a table of
three changes with a wall-clock number against each, and a note on which one actually mattered.

**I publish what doesn't work.** Every project has a *Known limitations* section. Triagent's opens
with "~50s per triage on CPU" and "5/8 on adversarial edge cases," and states plainly that it's a
**triage aid for a human, not an autopilot.** A benchmark you can't fail isn't a benchmark.

**I pick the boring tool when it's the right one.** Triagent parses logs with regex instead of
`drain3`, because template mining pays off when clustering thousands of near-identical streaming
lines — and here every input is a single one-off incident. Plain regex is more precise and drops
a dependency.

**I keep two implementations when they measure different things.** Triagent's app uses a merged
one-call prompt because it's twice as fast. Its *eval* uses an isolated severity prompt — because
if the model were reasoning about root cause in the same breath, the eval would no longer be
measuring the severity classifier on its own. Different jobs, different prompts, on purpose.

**I design for the system, not just the demo.** The slowest step in any "paste your error here"
tool is a human noticing the error. So the interesting endpoint is the webhook, not the dashboard.

---

## Toolkit

**Languages** · Python

**AI / ML** · Ollama · LangChain-style RAG pipelines · ChromaDB · sentence-transformers ·
`nomic-embed-text` · `llama3.2` / `llama3.1` · `moondream` (vision) · scikit-learn · XGBoost ·
Optuna · SHAP

**Backend & apps** · FastAPI · Uvicorn · Streamlit · Gradio · PyMuPDF · pytest

**Practices** · Local-first architecture · retrieval-augmented generation · hyperparameter search ·
model explainability · adversarial evaluation · performance profiling

---

<div align="center">

**Everything above installs with `pip install -r requirements.txt` and runs on your own hardware.**

No accounts. No keys. No billing.

</div>

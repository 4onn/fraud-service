# Lab 1: Refactor the notebook into a clean package

Lab 1 · 50 minutes · pairs · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m1) · [النسخة العربية](ar/lab-1.md)

Goal: the inherited code has a silent bug. Prove it with the golden file, then rebuild the scorer in layers until all 5,000 rows match.

- **Starting point:** tag `lab1-start`
- **Lab files:** [`labs/handouts/lab1/`](handouts/lab1/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-1.md, then teach me Lab 1 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.
```

## What this module teaches, in plain words

- Code that runs without an error can still give wrong answers. Check results against answers you already know are right.
- Keep the business rules apart from the tools that run them, so you can replace a tool without rewriting the rules.
- A predictable project layout lets anyone on the team find things and change them safely.
- Good structure is what makes everything later possible: testing, packaging and automation.

## The lab step by step

Prefer to drive each step yourself? Paste these one at a time instead of the prompt above.

### 1. Catch the legacy code lying

**What happens:** The old script runs without an error, yet about 182 of its scores are wrong. You compare its output with the trusted answers and find the one line that causes the difference.

```text
Step 1. Run `python legacy/notebook_v1.py`, then `python scripts/check_golden.py scored_notebook.csv`.
(If scripts/check_golden.py is missing, I will paste it from the lab page.)
About 182 rows should differ from the golden file. Look at the timestamps of the rows that differ,
compare the feature code in legacy/notebook_v1.py with training_features() in
scripts/make_course_assets.py, and show me the exact line that causes the skew.
Explain it in plain words. Do not fix the legacy script.
```

### 2. Domain layer

**What happens:** You write down the vocabulary of the problem (a transaction, a score, a decision) and the business rule, with no technical tools mixed in. One function builds the model's inputs, exactly the way training did.

```text
Step 2. Create src/fraud_service/domain/entities.py and src/fraud_service/domain/policies.py.
If I pasted the code from section 5 of the lab page below, save it exactly. Otherwise write it to this spec:
- entities.py: Channel and Decision (StrEnum), Transaction (pydantic), FeatureVector, RawScore, FraudScore.
  Transaction.to_features() must reproduce training_features() in scripts/make_course_assets.py exactly.
- policies.py: decide(score: RawScore, block_threshold: float) -> Decision, with a review band of 0.15
  below the block threshold.
Only standard-library and pydantic imports. Then walk me through every import and explain why this
layer must not import pandas, joblib or FastAPI.
```

### 3. Service layer

**What happens:** The service connects the pieces: it takes a transaction, asks a model for a score and applies the business rule. It talks to the model through a simple agreement, so any model can be plugged in later.

```text
Step 3. Create src/fraud_service/service/interfaces.py and src/fraud_service/service/scorer.py.
If I pasted the code from the lab page below, save it exactly. Otherwise:
- interfaces.py: a typing.Protocol named Model with an attribute model_version: str and a method
  predict_proba(features: FeatureVector) -> RawScore.
- scorer.py: a dataclass FraudScorer(model: Model, block_threshold: float) with
  score(txn: Transaction) -> FraudScore, which calls txn.to_features(), model.predict_proba() and decide().
Then show me that nothing under service/ imports sklearn, pandas or joblib.
```

### 4. Adapter

**What happens:** The adapter is the only place that knows about scikit-learn and the model file. If the model technology changes one day, only this file changes.

```text
Step 4. Create src/fraud_service/adapters/sklearn_model.py.
If I pasted the code from the lab page below, save it exactly. Otherwise: a class
SklearnModel(pipeline, model_version) with a classmethod load(path) that reads the joblib bundle
{"pipeline": ..., "version": ...}, and predict_proba(features) that builds a one-row pandas DataFrame from
features.values and returns a RawScore.
It must satisfy the Model protocol without inheriting from it. Explain why that works in Python.
```

### 5. Batch entrypoint and the golden check

**What happens:** The entrypoint wires the real pieces together and scores all 5,000 transactions. Then you compare with the trusted answers: every single row must match.

```text
Step 5. Create src/fraud_service/batch.py, the composition root (or save the version I pasted from the lab page).
It loads SklearnModel from models/fraud_xgb_v3.joblib, builds FraudScorer(block_threshold=0.85),
reads data/transactions_sample.csv and writes scored.csv with the columns
transaction_id, score (9 decimals), decision, model_version. Use argparse with defaults for the options
--input, --output, --model and --threshold.
Then run `python -m fraud_service.batch` and `python scripts/check_golden.py scored.csv`.
The goal is 5,000/5,000 matching rows. If any row differs, find the cause in to_features();
do not touch the golden file.
```

### 6. Measure and lint

**What happens:** A quick quality check. The linter finds small mistakes, and the import timer shows that loading your code is now almost instant, because the model loads only when the program starts.

```text
Step 6. Run `ruff check src` and fix what it reports without changing behaviour.
Then run `python -X importtime -c "import fraud_service.service.scorer" 2>&1 | tail -1`
(on Windows PowerShell, run it without the `| tail -1` and read the last line).
Explain the number: why does importing the service no longer load the model or scikit-learn,
and why does that matter for tests and for start-up time?
```

### 7. Commit

**What happens:** You save a snapshot of your work with a clear message, so you can always return to this point.

```text
Step 7. Show me `git status` and summarise what changed in this lab in 3 bullets.
Check that no scored*.csv file and no .venv/ folder will be committed.
Then commit with the message "refactor: extract clean architecture layers from notebook". Do not push.
```

## If something breaks

```text
Something went wrong in Lab 1. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 1 starting point (tag `lab1-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

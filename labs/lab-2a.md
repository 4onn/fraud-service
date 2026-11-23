# Lab 2a: Build the prediction API

Lab 2a · 40 minutes · pairs · Day 1 · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m2) · [النسخة العربية](ar/lab-2a.md)

Goal: expose the Lab 1 scorer over HTTP with a strict request schema, a health endpoint and a readiness endpoint. The domain and service layers do not change by a single line.

- **Starting point:** tag `lab2a-start`
- **Lab files:** [`labs/handouts/lab2a/`](handouts/lab2a/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-2a.md, then teach me Lab 2a one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.
```

## What this module teaches, in plain words

- An API is a promise to everyone who uses your model: what they must send, and what they get back.
- Check every request at the door. A clear, early rejection costs far less than a wrong decision later.
- When something fails, say what went wrong in plain terms, and never reveal private data or internal details.
- Measure speed before and after a change, so an improvement is a fact and not a feeling.

## The lab step by step

Prefer to drive each step yourself? Paste these one at a time instead of the prompt above.

### 1. Settings and schemas

**What happens:** You write down, in one place, the settings the service reads (such as where the model is and the block threshold), and the exact shape of a valid request and response. Anything that does not fit the shape is rejected.

```text
Step 1. Create src/fraud_service/config.py and src/fraud_service/api/schemas.py.
If I pasted the code from the lab page below, save it exactly. Otherwise:
- config.py: Settings(BaseSettings) with env_prefix "FRAUD_" and env_file ".env"; fields model_path
  (default models/fraud_xgb_v3.joblib), block_threshold (0.85, between 0.5 and 0.99),
  log_level ("INFO") and git_sha ("dev").
- schemas.py: PredictRequest with extra="forbid" and limits on every field, plus a to_domain() method
  that returns a Transaction; PredictResponse; ErrorBody; ErrorEnvelope; HealthResponse.
Explain why the API schema is a separate class from the domain Transaction.
```

### 2. App factory, lifespan and middleware

**What happens:** The app loads the model once when it starts, not on every request, and gives every request a tracking number so any problem can be traced later.

```text
Step 2. Create src/fraud_service/api/app.py (use the version from the lab page if I pasted it below).
create_app() is an app factory with a lifespan that reads Settings, loads SklearnModel once and
stores a FraudScorer and the settings on app.state; a middleware gives every request a trace id
and returns it in a response header; the router is mounted under /v1.
Then explain the order of events when the server starts, and when one request arrives.
```

### 3. Routes

**What happens:** Three addresses: one scores a transaction, one answers “is the process alive?”, and one answers “is it ready to take traffic?”.

```text
Step 3. Create src/fraud_service/api/routes.py (use the version from the lab page if I pasted it below):
- get_scorer(request): returns app.state.scorer, or raises 503 with the header Retry-After: 5.
- POST /predict: a plain def (not async def) that takes PredictRequest, returns PredictResponse
  with the trace id, and sets the X-Model-Version header.
- GET /health: liveness, no I/O. GET /ready: 503 until the scorer exists.
Explain why /predict is a plain def, and the difference between health and readiness.
```

### 4. Run it and try to break it

**What happens:** You start the service and send it good and bad requests. A good request gets a score; a bad one gets a clear rejection that names the problem.

```text
Step 4. Start the server with
uvicorn fraud_service.api.app:create_app --factory --reload
(in the background, or tell me to run it in a second terminal). Then send the five requests from the lab:
payloads/sample.json, payloads/malformed/amount_negative.json and payloads/malformed/typo_amount_sr.json
to POST /v1/predict, then GET /v1/health and GET /v1/ready. On Windows PowerShell use curl.exe.
For each response, explain in one line what it shows. Then open http://localhost:8000/docs and tell me
where that page came from.
```

### 5. Look closely at the error, then commit

**What happens:** You notice that the default error message repeats what the customer sent. That is harmless for an amount but dangerous for a national ID. You fix it in Lab 2b; for now you save your work.

```text
Step 5. Look at the 422 response for typo_amount_sr.json and tell me what private data that error
body would leak if a field held a national ID number. Do not fix it yet: that is Lab 2b.
Then commit with the message "feat(api): prediction endpoint with validation, health and readiness".
```

## If something breaks

```text
Something went wrong in Lab 2a. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 2a starting point (tag `lab2a-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

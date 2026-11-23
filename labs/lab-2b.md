# Lab 2b: Harden the API

Lab 2b · 35 minutes · pairs · Day 2 · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m2) · [النسخة العربية](ar/lab-2b.md)

Goal: warm the model up at startup, give every failure the same error envelope, and measure the service under load.

- **Starting point:** tag `lab2b-start`
- **Lab files:** [`labs/handouts/lab2b/`](handouts/lab2b/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-2b.md, then teach me Lab 2b one step at a time.
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

### 1. Measure first

**What happens:** Before changing anything, you time the first three requests after a restart. These numbers are your “before” picture.

```text
Step 1. Restart the server, then time the first three requests to POST /v1/predict with
payloads/sample.json (the lab uses curl -s -o /dev/null -w "%{time_total}s\n").
Write the three numbers into a new BENCHMARKS.md under the heading "Lab 2b: before warm-up",
and explain why the first request is slower than the others.
```

### 2. Warm-up and one error envelope

**What happens:** The service now makes one practice prediction while it starts, so the first real customer does not wait. Every error, of any kind, now has the same shape and never repeats the customer's data back.

```text
Step 2. Replace src/fraud_service/api/app.py with the hardened version from the lab (pasted below, if available).
It must: run one warm-up prediction in the lifespan before the app reports ready; add exception
handlers for RequestValidationError, HTTPException and any unhandled Exception; build every error
body with one _error() helper as {"error": {"code", "message", "trace_id", "details"}}; never include
the rejected "input" value.
Then restart, repeat step 1, send payloads/malformed/amount_negative.json again, and compare the
error body with the one from Lab 2a.
```

### 3. Prove that a crash leaks nothing

**What happens:** You cause a deliberate crash with a fake password in its message, and check that the customer sees only a generic error and a tracking number, while the details stay in the server log.

```text
Step 3. Add this temporary route to src/fraud_service/api/routes.py:

@router.get("/boom")
def boom():
    raise RuntimeError("database password is hunter2")

Call GET /v1/boom. Show me that the response contains only INTERNAL_ERROR and a trace id, and that
the message and the stack trace appear only in the server log. Then delete the route and confirm it is gone.
```

### 4. Load test

**What happens:** You send 2,000 requests, 25 at a time, and record how fast the service answers. Then you try four worker processes and see which number changes.

```text
Step 4. Save scripts/loadtest.py from the lab (pasted below, if available) and run `python scripts/loadtest.py`.
Record p50, p99 and throughput in BENCHMARKS.md. Then restart the server with the option
--workers 4 (and without --reload), run it again, record the numbers, and explain in plain words which number
moved and why.
Optional: run the async trap from the lab and explain why blocking code inside async def stalls the whole server.
```

### 5. Commit

**What happens:** You check that the temporary crash route is gone and save your work.

```text
Step 5. Confirm that the /boom route is gone, show me git status, then commit with the message
"feat(api): warm-up, error envelope and first benchmarks".
```

## If something breaks

```text
Something went wrong in Lab 2b. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 2b starting point (tag `lab2b-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

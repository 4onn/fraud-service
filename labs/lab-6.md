# Lab 6: Harden config, secrets and logs

Lab 6 · 30 minutes · pairs · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m6) · [النسخة العربية](ar/lab-6.md)

Goal: typed settings that fail fast, JSON logs you can query, and one rehearsal of the secret-leak response. All of it travels through a pull request.

- **Starting point:** tag `lab6-start`
- **Lab files:** [`labs/handouts/lab6/`](handouts/lab6/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-6.md, then teach me Lab 6 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.
```

## What this module teaches, in plain words

- Keep settings outside the code and check them at start-up. A service that refuses to start with a clear message beats one that breaks hours later.
- Secrets never go into code or shared files. If one leaks, replace it at once: deleting it afterwards is not enough.
- Write logs a machine can search, so you can answer “what happened to this request?” in seconds.
- Most real outages come from a wrong setting, not from bad code.

## The lab step by step

Prefer to drive each step yourself? Paste these one at a time instead of the prompt above.

### 1. Typed settings

**What happens:** The service now checks its own settings when it starts. If the model file is missing, it stops at once with a clear message instead of failing later.

```text
Step 1. Replace src/fraud_service/config.py with the Lab 6 version and add configs/settings.example.env (both
pasted below, if available). model_path is now required and validated at start-up.
Create a local .env containing FRAUD_MODEL_PATH=models/fraud_xgb_v3.joblib and confirm that git ignores it.
Add tests/unit/test_config.py from the lab and run pytest -m unit.
```

### 2. Watch it fail fast

**What happens:** You start the service with a wrong setting on purpose and see it refuse to start, with a message that tells the operator exactly what to fix.

```text
Step 2. Start the server with a wrong model path:
FRAUD_MODEL_PATH=/nope uvicorn fraud_service.api.app:create_app --factory
(on Windows PowerShell: $env:FRAUD_MODEL_PATH="/nope", start the server, then Remove-Item Env:FRAUD_MODEL_PATH).
Show me the error and explain why refusing to start is better than a 500 error three hours after a deployment.
```

### 3. JSON logs you can query

**What happens:** Every log line is now a small record a computer can search. You answer real questions (how fast are we? what happened to this one request?) straight from the log file.

```text
Step 3. Add src/fraud_service/logging_setup.py and replace api/app.py and api/routes.py with the final
versions from Module 6 (pasted below, if available). Start the server with --no-access-log and send
standard output to service.log, run python scripts/loadtest.py -n 300 -c 10, and send one request with
the header X-Trace-Id: my-trace-1.
Then answer from service.log only: the median scoring latency, and every event for trace my-trace-1.
Use jq if it is installed, otherwise Python.
```

### 4. Secret-leak drill

**What happens:** You “leak” a fake token, delete it, and discover it is still in the history. Then you write the right response in the right order: replace the token first, then clean up, then add a scanner.

```text
Step 4. Secret-leak drill. Do not push during this step. First commit my work so far. Then:
1. Append FRAUD_REGISTRY_TOKEN=ghp_THIS_IS_A_FAKE_TOKEN_FOR_THE_DRILL to configs/dev.env and commit "chore: tune dev config".
2. Delete that line and commit "chore: remove token".
3. Run git log -p --all -S "ghp_THIS_IS" --oneline and show me that the token is still in the history.
Help me write INCIDENT.md with the response in order: rotate the token now, clean the history, add a scanner.
Then drop the two drill commits with git reset --hard HEAD~2 and explain why that is safe only because
they were never pushed.
```

### 5. Check the mask, then open a pull request

**What happens:** You check that secrets written to the log are hidden, then send all of this work through a pull request, where the pipeline checks it once more.

```text
Step 5. Temporarily add log.info("debug", token="abc123") to the predict route, call it once, and show me
the line in service.log where the token appears as ***MASKED***. Remove the line.
Run pytest -m "not slow", commit with "feat: typed settings, structured JSON logs, secret hygiene",
push the branch harden-config-logging, and tell me how to open the pull request and merge it when the checks are green.
```

## If something breaks

```text
Something went wrong in Lab 6. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 6 starting point (tag `lab6-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

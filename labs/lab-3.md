# Lab 3: Containerise the fraud service

Lab 3 · 55 minutes · pairs · split by the break · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m3) · [النسخة العربية](ar/lab-3.md)

Goal: a slim, non-root, healthchecked image and a compose stack, with your own measurements of size, build time and latency.

- **Starting point:** tag `lab3-start`
- **Lab files:** [`labs/handouts/lab3/`](handouts/lab3/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-3.md, then teach me Lab 3 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.
```

## What this module teaches, in plain words

- Package the service with everything it needs, so it runs the same way on every machine. No more “it works on my laptop”.
- A smaller, cleaner package is faster to ship and safer to run.
- Never put passwords or keys inside what you ship: anyone who receives the package can read them.
- Run the whole system on your own laptop the way it will run in production, before anyone depends on it.

## The lab step by step

Prefer to drive each step yourself? Paste these one at a time instead of the prompt above.

### 1. Lock the dependencies

**What happens:** You write down the exact version of every library the service needs, for the Linux system inside the container, so every build installs exactly the same thing.

```text
Step 1. Run:
pip install uv
uv pip compile pyproject.toml --python-version 3.12 --python-platform linux --no-annotate --no-header -o requirements.lock
Explain what requirements.lock contains, why it lists packages I never asked for by name, and why we
resolve for Linux even though my laptop may run Windows or macOS.
```

### 2. Build the naive image and plant a fake secret

**What happens:** You first build the image the way most beginners do, and plant a fake secret, to see the problems with your own eyes.

```text
Step 2. Save Dockerfile.naive from the lab (pasted below, if available). Create the fake secret with
echo "FAKE_API_KEY=sk-not-a-real-key-123" > leaked.env
and build with
time docker build -f Dockerfile.naive -t fraud-service:naive .
(on Windows PowerShell, drop the word time and use Measure-Command if you want the duration).
While it builds, explain what "transferring context" means and why its size matters.
```

### 3. Look at what you built

**What happens:** You check the size, find the fake secret inside the image, and see that the service runs as the all-powerful root user.

```text
Step 3. Run:
docker images fraud-service
docker run --rm fraud-service:naive cat leaked.env
docker run --rm fraud-service:naive whoami
Record the image size and the build time in BENCHMARKS.md, and explain the two security problems
these commands prove.
```

### 4. Ignore file and the real Dockerfile

**What happens:** A list of things Docker must never see, and a two-stage recipe: it builds in one room and ships only what is needed from another. Smaller, faster and safer.

```text
Step 4. Create .dockerignore and the multi-stage Dockerfile from the lab (pasted below, if available).
The Dockerfile installs requirements.lock into /opt/venv in a builder stage; the python:3.12-slim
runtime stage copies only /opt/venv, the model and src/, runs as the non-root user appuser, has a
HEALTHCHECK on /v1/ready and starts uvicorn with the app factory.
Build with:
time docker build -t fraud-service:dev .
Then run docker images fraud-service, whoami and ls /app in the new image, confirm that leaked.env
is not there, and record the new numbers.
```

### 5. Prove the cache works

**What happens:** You change one line of code and rebuild. Because the heavy parts did not change, Docker reuses them and the rebuild takes seconds.

```text
Step 5. Add a comment to src/fraud_service/api/routes.py and rebuild with
time docker build -t fraud-service:dev .
Show me which steps say CACHED, record the rebuild time, and explain why the order of the
instructions in the Dockerfile makes this rebuild take seconds.
```

### 6. Compose stack

**What happens:** One command starts the whole system, your service plus a cache, the way production would run it, with health checks and a memory limit.

```text
Step 6. Create configs/dev.env and docker-compose.yml from the lab (pasted below, if available). Then run:
docker compose up --build -d
docker compose ps        # wait until fraud-api shows (healthy)
and call GET /v1/ready and POST /v1/predict with payloads/sample.json.
Explain what depends_on with condition: service_healthy does, and why configs/dev.env holds no secrets.
```

### 7. Measure, clean up and commit

**What happens:** You measure speed and memory inside the container, check that it shuts down gracefully, delete the deliberately bad files and save your work.

```text
Step 7. Run python scripts/loadtest.py against the container and compare p99 with my Lab 2b number,
run docker stats --no-stream, and time docker compose stop fraud-api. Record all three in BENCHMARKS.md.
Then run docker compose down, delete leaked.env and Dockerfile.naive, show me git status, and commit
with "feat(docker): multi-stage image, healthcheck and compose stack".
```

## If something breaks

```text
Something went wrong in Lab 3. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 3 starting point (tag `lab3-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

# Lab 5: Build the pipeline

Lab 5 · 45 minutes · pairs, one repository each · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m5) · [النسخة العربية](ar/lab-5.md)

Goal: a working pipeline on your own GitHub repository, branch protection switched on, and one deliberately bad pull request blocked, diagnosed and fixed.

- **Starting point:** tag `lab5-start`
- **Lab files:** [`labs/handouts/lab5/`](handouts/lab5/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-5.md, then teach me Lab 5 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.
```

## What this module teaches, in plain words

- Let a machine check every change the same way, every time. People forget steps; automation does not.
- Make the checks mandatory: no change reaches the main version until it passes them.
- Fix the cause of a failure. Never weaken a check just to get a green light.
- Every run leaves a record, which becomes ready-made evidence for reviewers and auditors.

## The lab step by step

Prefer to drive each step yourself? Paste these one at a time instead of the prompt above.

### 1. Check locally, then add the workflow

**What happens:** You run on your laptop the same checks the pipeline will run, so you fix problems in seconds instead of waiting minutes. Then you add the pipeline files and push.

```text
Step 1. Run the four local gates and fix anything red without weakening a check:
ruff check src tests
mypy src/fraud_service
lint-imports
pytest -m "not slow" --cov --cov-fail-under=80
Then create .github/workflows/ci.yml and .github/workflows/nightly.yml from the lab (pasted below, if
available), explain each job in one line, commit with "ci: lint, test, image smoke test and publish" and push.
```

### 2. Find your image

**What happens:** When the pipeline passes on main, it publishes your container image to GitHub. You download it the way any user would.

```text
Step 2. My first run on main is green. Give me the docker pull command for my image on ghcr.io (my GitHub
user name in lower case, tag main), and tell me how to make the package public in the
GitHub website if the pull is refused as unauthorized.
```

### 3. Switch on branch protection

**What happens:** You switch on a rule in GitHub: nothing enters the main version unless the checks pass. You do this in the website; the agent explains each click.

```text
Step 3. Walk me through switching on branch protection for main in the GitHub website: require a
pull request, and require the status checks lint, test and image-smoke. Explain why the checks only
appear in the search box after they have run once.
```

### 4. Open a bad pull request

**What happens:** You make two “small improvements” that are really mistakes, open a pull request, and watch the pipeline refuse it.

```text
Step 4. This is a deliberate training exercise. Create a branch named bad-pr and make exactly these two changes:
1. In src/fraud_service/service/scorer.py add the line:
   from fraud_service.api.schemas import PredictRequest  # noqa: F401
2. In src/fraud_service/domain/entities.py, inside to_features(), change hour < 6 to hour <= 6.
Commit with "feat: small improvements to scorer and features", push the branch, and tell me how to open
the pull request. When the checks fail, read the logs with me and explain which job caught which change, and why.
```

### 5. Fix it properly

**What happens:** You remove both mistakes on the same branch, without touching any test, wait for green checks, and merge.

```text
Step 5. On the same branch, undo both changes. Do not touch any test or the layer contract.
Commit and push. When all checks are green, tell me to merge the pull request in GitHub, then run git switch main
and git pull.
```

### 6. Record

**What happens:** You write down how long each check took on your first and latest runs, and see why the second image build was faster.

```text
Step 6. Help me add to BENCHMARKS.md the duration of each job for my first run and my latest run (I will read them
from the Actions tab). Then explain why the second image build is clearly faster.
```

## If something breaks

```text
Something went wrong in Lab 5. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 5 starting point (tag `lab5-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

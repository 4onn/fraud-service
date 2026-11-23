# Lab 4: Build the test suite

Lab 4 · 50 minutes · pairs · [Open this lab on the website](https://aie113.ibrahimalshehri.sa/#m4) · [النسخة العربية](ar/lab-4.md)

Goal: a three-level pytest suite: fast tests green in seconds with branch coverage above 80%, all 40 malformed payloads rejected, and behaviour tests passing against the real model.

- **Starting point:** tag `lab4-start`
- **Lab files:** [`labs/handouts/lab4/`](handouts/lab4/)

## Start with your agent

Open this folder in your coding agent (Claude Code, GitHub Copilot agent mode, Cursor, Codex…) and paste:

```text
Read AGENTS.md and labs/lab-4.md, then teach me Lab 4 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.
```

## What this module teaches, in plain words

- An AI system can be wrong while it looks perfectly healthy, so test what it decides, not only whether it runs.
- Many small, fast checks catch most mistakes in seconds; a few slow, thorough checks guard what matters most.
- Keep a set of trusted answers and compare against it on every change, so a silent drift is caught early.
- A failing test is good news: it found the problem before a customer did.

## The lab step by step

Prefer to drive each step yourself? Paste these one at a time instead of the prompt above.

### 1. Fixtures

**What happens:** Shared helpers for every test: a fake model that always returns a known score, a sample transaction, and a way to start the app with the fake model instead of the real one.

```text
Step 1. Create tests/conftest.py from the lab (pasted below, if available).
Then explain in one sentence how client_factory gets the fake model into the app (which function
it overrides), and why the real_model fixture is session-scoped.
```

### 2. Unit tests

**What happens:** Small checks for the business rule, the model inputs and the scorer. All of them together run in a fraction of a second.

```text
Step 2. Create tests/unit/test_policies.py, tests/unit/test_features.py and tests/unit/test_scorer.py from
the lab (pasted below, if available). Run pytest -m unit. Expected: 16 passed in well under a second.
For each file, tell me in one line which promise of the code it protects.
```

### 3. Integration tests

**What happens:** You test the service from the outside, the way a client app would use it: good requests succeed, and every one of the 40 bad requests is rejected.

```text
Step 3. Create tests/integration/test_predict_api.py from the lab (pasted below, if available).
Run pytest -m integration. Expected: 46 passed, 40 of them from one parametrised test over payloads/malformed/.
Explain what parametrisation buys us here.
```

### 4. Behaviour tests

**What happens:** These check the real model's decisions: the score must not change when only a name or an ID changes, a much larger amount must not look safer, and all 5,000 trusted answers must still come out the same.

```text
Step 4. Create tests/behavioural/test_model_behaviour.py from the lab (pasted below, if available). Run:
pytest -m "behavioural and not slow"     # expected: 5 passed, about 1 second
pytest -m slow                           # expected: the golden-file test, 1 passed, about 20 seconds
Explain in plain words how these tests differ from the unit tests.
```

### 5. Break it on purpose

**What happens:** You put the old bug back on purpose and watch which test catches it. This proves that your tests really protect you.

```text
Step 5. In src/fraud_service/domain/entities.py, change hour < 6 to hour <= 6.
Run pytest -m "not slow", then pytest -m slow. Tell me which test catches the bug and at which level
of the testing pyramid. Then undo the change and confirm that git diff is empty.
```

### 6. Coverage, then commit

**What happens:** You see which lines of code no test ever runs, pick the riskiest one, and save your work.

```text
Step 6. Run pytest -m "not slow" --cov --cov-report=term-missing --cov-fail-under=80.
Read the Missing column with me: pick the uncovered line that is riskiest (not the easiest) and
describe the test that would cover it. Record the suite time in BENCHMARKS.md, then commit with
"test: three-level suite with behavioural tripwires".
```

## If something breaks

```text
Something went wrong in Lab 4. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.
```

## Joining late or stuck?

Ask your agent: "Bring my project to the Lab 4 starting point (tag `lab4-start`), keep my own notes, and explain what changed." The steps are in AGENTS.md.

[All labs](../README.md)

# المختبر 5: بناء خط الأتمتة

المختبر 5 · 45 دقيقة · في أزواج، ولكل متدرب مستودعه · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m5) · [English version](../lab-5.md)

الهدف: خط أتمتة يعمل على مستودعك في GitHub، وحماية الفرع مفعّلة، وطلب دمج سيئ متعمَّد يُمنع ويُشخَّص ويُصلَح.

- **نقطة البداية:** tag `lab5-start`
- **ملفات المختبر:** [`labs/handouts/lab5/`](../handouts/lab5/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-5.md, then teach me Lab 5 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.

Reply in Arabic. Keep code, file names and commands in English.
```

## ما تعلّمه هذه الوحدة بكلمات بسيطة

- دع الآلة تفحص كل تغيير بالطريقة نفسها في كل مرة. البشر ينسون الخطوات، والأتمتة لا تنسى.
- اجعل الفحوص إلزامية: لا يصل أي تغيير إلى النسخة الرئيسية حتى ينجح فيها.
- عالج سبب الفشل، ولا تُضعف الفحص أبدًا لمجرد الحصول على الضوء الأخضر.
- كل تشغيل يترك سجلًا، يصبح دليلًا جاهزًا للمراجعين والمدققين.

## المختبر خطوة بخطوة

تفضّل أن تقود كل خطوة بنفسك؟ الصق هذه الأوامر واحدًا تلو الآخر بدل الأمر السابق.

### 1. افحص محليًا ثم أضف سير العمل

**ما الذي يحدث:** تشغّل على جهازك الفحوص نفسها التي سيشغّلها خط الأتمتة، فتصلح المشكلات في ثوانٍ بدل الانتظار دقائق. ثم تضيف ملفات خط الأتمتة وترفعها.

```text
Step 1. Run the four local gates and fix anything red without weakening a check:
ruff check src tests
mypy src/fraud_service
lint-imports
pytest -m "not slow" --cov --cov-fail-under=80
Then create .github/workflows/ci.yml and .github/workflows/nightly.yml from the lab (pasted below, if
available), explain each job in one line, commit with "ci: lint, test, image smoke test and publish" and push.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. اعثر على صورتك

**ما الذي يحدث:** عندما ينجح خط الأتمتة على الفرع الرئيسي، ينشر صورة الحاوية على GitHub. وتنزّلها كما يفعل أي مستخدم.

```text
Step 2. My first run on main is green. Give me the docker pull command for my image on ghcr.io (my GitHub
user name in lower case, tag main), and tell me how to make the package public in the
GitHub website if the pull is refused as unauthorized.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. فعّل حماية الفرع

**ما الذي يحدث:** تفعّل قاعدة في GitHub: لا يدخل شيء إلى النسخة الرئيسية ما لم تنجح الفحوص. تقوم بذلك من الموقع بنفسك، ويشرح لك الوكيل كل خطوة.

```text
Step 3. Walk me through switching on branch protection for main in the GitHub website: require a
pull request, and require the status checks lint, test and image-smoke. Explain why the checks only
appear in the search box after they have run once.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. افتح طلب دمج سيئًا

**ما الذي يحدث:** تُجري «تحسينين صغيرين» هما في الحقيقة خطآن، وتفتح طلب دمج، وتشاهد خط الأتمتة يرفضه.

```text
Step 4. This is a deliberate training exercise. Create a branch named bad-pr and make exactly these two changes:
1. In src/fraud_service/service/scorer.py add the line:
   from fraud_service.api.schemas import PredictRequest  # noqa: F401
2. In src/fraud_service/domain/entities.py, inside to_features(), change hour < 6 to hour <= 6.
Commit with "feat: small improvements to scorer and features", push the branch, and tell me how to open
the pull request. When the checks fail, read the logs with me and explain which job caught which change, and why.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. أصلح المشكلة بالطريقة الصحيحة

**ما الذي يحدث:** تزيل الخطأين في الفرع نفسه دون المساس بأي اختبار، وتنتظر نجاح الفحوص، ثم تدمج.

```text
Step 5. On the same branch, undo both changes. Do not touch any test or the layer contract.
Commit and push. When all checks are green, tell me to merge the pull request in GitHub, then run git switch main
and git pull.

Reply in Arabic. Keep code, file names and commands in English.
```

### 6. سجّل النتائج

**ما الذي يحدث:** تسجّل مدة كل فحص في أول تشغيل وآخر تشغيل، وتفهم لماذا كان بناء الصورة الثاني أسرع.

```text
Step 6. Help me add to BENCHMARKS.md the duration of each job for my first run and my latest run (I will read them
from the Actions tab). Then explain why the second image build is clearly faster.

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 5. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 5 starting point (tag `lab5-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

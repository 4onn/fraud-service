# المختبر 6: تحصين الإعدادات والأسرار والسجلات

المختبر 6 · 30 دقيقة · عمل ثنائي · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m6) · [English version](../lab-6.md)

الهدف: إعدادات محددة الأنواع تطبّق الفشل المبكر (fail fast)، وسجلات JSON تستطيع الاستعلام عنها، وتمرين واحد على الاستجابة لتسريب سر. وكل ذلك يمر عبر طلب دمج (pull request).

- **نقطة البداية:** tag `lab6-start`
- **ملفات المختبر:** [`labs/handouts/lab6/`](../handouts/lab6/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-6.md, then teach me Lab 6 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.

Reply in Arabic. Keep code, file names and commands in English.
```

## ما تعلّمه هذه الوحدة بكلمات بسيطة

- أبقِ الإعدادات خارج الكود وافحصها عند التشغيل. الخدمة التي ترفض العمل برسالة واضحة أفضل من خدمة تتعطل بعد ساعات.
- الأسرار لا توضع أبدًا في الكود أو في الملفات المشتركة. وإن تسرّب سرّ فاستبدله فورًا، فحذفه لاحقًا لا يكفي.
- اكتب سجلات يستطيع الحاسب البحث فيها، لتجيب في ثوانٍ عن سؤال: ماذا حدث لهذا الطلب؟
- معظم الأعطال الحقيقية سببها إعداد خاطئ، لا كود سيئ.

## المختبر خطوة بخطوة

تفضّل أن تقود كل خطوة بنفسك؟ الصق هذه الأوامر واحدًا تلو الآخر بدل الأمر السابق.

### 1. إعدادات محددة الأنواع

**ما الذي يحدث:** أصبحت الخدمة تفحص إعداداتها بنفسها عند بدء التشغيل. فإذا كان ملف النموذج مفقودًا توقفت فورًا برسالة واضحة، بدل أن تتعطل لاحقًا.

```text
Step 1. Replace src/fraud_service/config.py with the Lab 6 version and add configs/settings.example.env (both
pasted below, if available). model_path is now required and validated at start-up.
Create a local .env containing FRAUD_MODEL_PATH=models/fraud_xgb_v3.joblib and confirm that git ignores it.
Add tests/unit/test_config.py from the lab and run pytest -m unit.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. شاهدها تفشل مبكرًا

**ما الذي يحدث:** تشغّل الخدمة بإعداد خاطئ عمدًا، وتراها ترفض العمل برسالة تخبر المشغّل بالضبط ما الذي يجب إصلاحه.

```text
Step 2. Start the server with a wrong model path:
FRAUD_MODEL_PATH=/nope uvicorn fraud_service.api.app:create_app --factory
(on Windows PowerShell: $env:FRAUD_MODEL_PATH="/nope", start the server, then Remove-Item Env:FRAUD_MODEL_PATH).
Show me the error and explain why refusing to start is better than a 500 error three hours after a deployment.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. سجلات JSON قابلة للاستعلام

**ما الذي يحدث:** أصبح كل سطر في السجل سجلًا صغيرًا يستطيع الحاسب البحث فيه. وتجيب عن أسئلة حقيقية (ما سرعتنا؟ ماذا حدث لهذا الطلب تحديدًا؟) من ملف السجل مباشرة.

```text
Step 3. Add src/fraud_service/logging_setup.py and replace api/app.py and api/routes.py with the final
versions from Module 6 (pasted below, if available). Start the server with --no-access-log and send
standard output to service.log, run python scripts/loadtest.py -n 300 -c 10, and send one request with
the header X-Trace-Id: my-trace-1.
Then answer from service.log only: the median scoring latency, and every event for trace my-trace-1.
Use jq if it is installed, otherwise Python.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. تمرين تسرّب سر

**ما الذي يحدث:** تُسرّب رمزًا وهميًا، ثم تحذفه، فتكتشف أنه ما زال في السجل التاريخي. ثم تكتب الاستجابة الصحيحة بالترتيب الصحيح: استبدل الرمز أولًا، ثم نظّف السجل، ثم أضف أداة فحص.

```text
Step 4. Secret-leak drill. Do not push during this step. First commit my work so far. Then:
1. Append FRAUD_REGISTRY_TOKEN=ghp_THIS_IS_A_FAKE_TOKEN_FOR_THE_DRILL to configs/dev.env and commit "chore: tune dev config".
2. Delete that line and commit "chore: remove token".
3. Run git log -p --all -S "ghp_THIS_IS" --oneline and show me that the token is still in the history.
Help me write INCIDENT.md with the response in order: rotate the token now, clean the history, add a scanner.
Then drop the two drill commits with git reset --hard HEAD~2 and explain why that is safe only because
they were never pushed.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. تحقق من الإخفاء ثم افتح طلب دمج

**ما الذي يحدث:** تتأكد أن الأسرار المكتوبة في السجل تظهر مخفية، ثم ترسل هذا العمل كله عبر طلب دمج، حيث يفحصه خط الأتمتة مرة أخرى.

```text
Step 5. Temporarily add log.info("debug", token="abc123") to the predict route, call it once, and show me
the line in service.log where the token appears as ***MASKED***. Remove the line.
Run pytest -m "not slow", commit with "feat: typed settings, structured JSON logs, secret hygiene",
push the branch harden-config-logging, and tell me how to open the pull request and merge it when the checks are green.

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 6. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 6 starting point (tag `lab6-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

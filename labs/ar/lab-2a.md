# المختبر 2أ: بناء واجهة التنبؤ

المختبر 2أ · 40 دقيقة · عمل ثنائي · اليوم 1 · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m2) · [English version](../lab-2a.md)

الهدف: إتاحة المقيِّم الذي بنيته في المختبر 1 عبر HTTP، مع مخطط طلب صارم، ونقطة نهاية للسلامة وأخرى للجاهزية. طبقتا النطاق والخدمة لا تتغيران بسطر واحد.

- **نقطة البداية:** tag `lab2a-start`
- **ملفات المختبر:** [`labs/handouts/lab2a/`](../handouts/lab2a/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-2a.md, then teach me Lab 2a one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.

Reply in Arabic. Keep code, file names and commands in English.
```

## ما تعلّمه هذه الوحدة بكلمات بسيطة

- الواجهة البرمجية وعدٌ لكل من يستخدم نموذجك: ما الذي يجب أن يرسله، وما الذي سيستلمه.
- افحص كل طلب عند الباب. الرفض الواضح المبكر أرخص بكثير من قرار خاطئ لاحقًا.
- عند حدوث خطأ، وضّح ما حدث بلغة بسيطة، ولا تكشف أبدًا بيانات خاصة أو تفاصيل داخلية.
- قِس السرعة قبل التغيير وبعده، ليكون التحسن حقيقة مثبتة لا مجرد انطباع.

## المختبر خطوة بخطوة

تفضّل أن تقود كل خطوة بنفسك؟ الصق هذه الأوامر واحدًا تلو الآخر بدل الأمر السابق.

### 1. الإعدادات والمخططات

**ما الذي يحدث:** تكتب في مكان واحد الإعدادات التي تقرؤها الخدمة (مثل مكان النموذج وحد الحظر)، والشكل الدقيق للطلب والاستجابة الصحيحين. وأي طلب لا يطابق هذا الشكل يُرفض.

```text
Step 1. Create src/fraud_service/config.py and src/fraud_service/api/schemas.py.
If I pasted the code from the lab page below, save it exactly. Otherwise:
- config.py: Settings(BaseSettings) with env_prefix "FRAUD_" and env_file ".env"; fields model_path
  (default models/fraud_xgb_v3.joblib), block_threshold (0.85, between 0.5 and 0.99),
  log_level ("INFO") and git_sha ("dev").
- schemas.py: PredictRequest with extra="forbid" and limits on every field, plus a to_domain() method
  that returns a Transaction; PredictResponse; ErrorBody; ErrorEnvelope; HealthResponse.
Explain why the API schema is a separate class from the domain Transaction.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. مصنع التطبيق ودورة حياته

**ما الذي يحدث:** يحمّل التطبيق النموذج مرة واحدة عند بدء التشغيل، لا مع كل طلب، ويعطي كل طلب رقم تتبع حتى يمكن تتبع أي مشكلة لاحقًا.

```text
Step 2. Create src/fraud_service/api/app.py (use the version from the lab page if I pasted it below).
create_app() is an app factory with a lifespan that reads Settings, loads SklearnModel once and
stores a FraudScorer and the settings on app.state; a middleware gives every request a trace id
and returns it in a response header; the router is mounted under /v1.
Then explain the order of events when the server starts, and when one request arrives.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. المسارات

**ما الذي يحدث:** ثلاثة عناوين: الأول يحسب درجة العملية، والثاني يجيب عن سؤال «هل الخدمة حية؟»، والثالث يجيب عن سؤال «هل هي جاهزة لاستقبال الطلبات؟».

```text
Step 3. Create src/fraud_service/api/routes.py (use the version from the lab page if I pasted it below):
- get_scorer(request): returns app.state.scorer, or raises 503 with the header Retry-After: 5.
- POST /predict: a plain def (not async def) that takes PredictRequest, returns PredictResponse
  with the trace id, and sets the X-Model-Version header.
- GET /health: liveness, no I/O. GET /ready: 503 until the scorer exists.
Explain why /predict is a plain def, and the difference between health and readiness.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. شغّل الخدمة وحاول كسرها

**ما الذي يحدث:** تشغّل الخدمة وترسل لها طلبات صحيحة وأخرى خاطئة. الطلب الصحيح يحصل على درجة، والخاطئ يُرفض برسالة واضحة تسمّي المشكلة.

```text
Step 4. Start the server with
uvicorn fraud_service.api.app:create_app --factory --reload
(in the background, or tell me to run it in a second terminal). Then send the five requests from the lab:
payloads/sample.json, payloads/malformed/amount_negative.json and payloads/malformed/typo_amount_sr.json
to POST /v1/predict, then GET /v1/health and GET /v1/ready. On Windows PowerShell use curl.exe.
For each response, explain in one line what it shows. Then open http://localhost:8000/docs and tell me
where that page came from.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. تمعّن في رسالة الخطأ ثم احفظ عملك

**ما الذي يحدث:** تلاحظ أن رسالة الخطأ الافتراضية تعيد ما أرسله العميل. هذا لا يضر في حالة المبلغ، لكنه خطير إن كان رقم هوية. ستعالج ذلك في المختبر 2ب، والآن تحفظ عملك.

```text
Step 5. Look at the 422 response for typo_amount_sr.json and tell me what private data that error
body would leak if a field held a national ID number. Do not fix it yet: that is Lab 2b.
Then commit with the message "feat(api): prediction endpoint with validation, health and readiness".

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 2a. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 2a starting point (tag `lab2a-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

# المختبر 2ب: تقوية الواجهة

المختبر 2ب · 35 دقيقة · عمل ثنائي · اليوم 2 · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m2) · [English version](../lab-2b.md)

الهدف: إحماء النموذج عند بدء التشغيل، ومنح كل فشل غلاف الخطأ نفسه، وقياس أداء الخدمة تحت الحمل.

- **نقطة البداية:** tag `lab2b-start`
- **ملفات المختبر:** [`labs/handouts/lab2b/`](../handouts/lab2b/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-2b.md, then teach me Lab 2b one step at a time.
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

### 1. قِس أولًا

**ما الذي يحدث:** قبل أي تغيير، تقيس زمن أول ثلاثة طلبات بعد إعادة التشغيل. هذه الأرقام هي صورة «قبل».

```text
Step 1. Restart the server, then time the first three requests to POST /v1/predict with
payloads/sample.json (the lab uses curl -s -o /dev/null -w "%{time_total}s\n").
Write the three numbers into a new BENCHMARKS.md under the heading "Lab 2b: before warm-up",
and explain why the first request is slower than the others.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. التسخين وشكل موحد للأخطاء

**ما الذي يحدث:** تُجري الخدمة الآن تنبؤًا تجريبيًا واحدًا أثناء بدء التشغيل، فلا ينتظر أول عميل حقيقي. وكل خطأ، أيًّا كان نوعه، أصبح له شكل واحد، ولا يعيد بيانات العميل أبدًا.

```text
Step 2. Replace src/fraud_service/api/app.py with the hardened version from the lab (pasted below, if available).
It must: run one warm-up prediction in the lifespan before the app reports ready; add exception
handlers for RequestValidationError, HTTPException and any unhandled Exception; build every error
body with one _error() helper as {"error": {"code", "message", "trace_id", "details"}}; never include
the rejected "input" value.
Then restart, repeat step 1, send payloads/malformed/amount_negative.json again, and compare the
error body with the one from Lab 2a.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. أثبت أن الانهيار لا يسرّب شيئًا

**ما الذي يحدث:** تتسبب عمدًا في انهيار يحمل كلمة مرور وهمية في رسالته، وتتأكد أن العميل لا يرى إلا خطأً عامًا ورقم تتبع، بينما تبقى التفاصيل في سجل الخادم فقط.

```text
Step 3. Add this temporary route to src/fraud_service/api/routes.py:

@router.get("/boom")
def boom():
    raise RuntimeError("database password is hunter2")

Call GET /v1/boom. Show me that the response contains only INTERNAL_ERROR and a trace id, and that
the message and the stack trace appear only in the server log. Then delete the route and confirm it is gone.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. اختبار التحمل

**ما الذي يحدث:** ترسل 2,000 طلب، 25 في كل مرة، وتسجل سرعة استجابة الخدمة. ثم تجرب أربع عمليات تشغيل (workers) وترى أي رقم يتغير.

```text
Step 4. Save scripts/loadtest.py from the lab (pasted below, if available) and run `python scripts/loadtest.py`.
Record p50, p99 and throughput in BENCHMARKS.md. Then restart the server with the option
--workers 4 (and without --reload), run it again, record the numbers, and explain in plain words which number
moved and why.
Optional: run the async trap from the lab and explain why blocking code inside async def stalls the whole server.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. احفظ عملك في Git

**ما الذي يحدث:** تتأكد من حذف مسار الانهيار المؤقت، ثم تحفظ عملك.

```text
Step 5. Confirm that the /boom route is gone, show me git status, then commit with the message
"feat(api): warm-up, error envelope and first benchmarks".

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 2b. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 2b starting point (tag `lab2b-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

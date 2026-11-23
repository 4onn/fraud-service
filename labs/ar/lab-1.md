# المختبر 1: إعادة هيكلة الدفتر إلى حزمة نظيفة

المختبر 1 · 50 دقيقة · عمل ثنائي · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m1) · [English version](../lab-1.md)

الهدف: في الكود الموروث خلل صامت. أثبته باستخدام الملف المرجعي (golden file)، ثم أعد بناء المُقيِّم في طبقات حتى تتطابق الصفوف الـ5,000 كلها.

- **نقطة البداية:** tag `lab1-start`
- **ملفات المختبر:** [`labs/handouts/lab1/`](../handouts/lab1/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-1.md, then teach me Lab 1 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.

Reply in Arabic. Keep code, file names and commands in English.
```

## ما تعلّمه هذه الوحدة بكلمات بسيطة

- الكود الذي يعمل دون أخطاء قد يعطي نتائج خاطئة. قارن النتائج دائمًا بإجابات تعرف مسبقًا أنها صحيحة.
- افصل قواعد العمل عن الأدوات التي تنفذها، لتتمكن من استبدال أداة دون إعادة كتابة القواعد.
- الترتيب الواضح والمتوقع للمشروع يساعد أي شخص في الفريق على إيجاد ما يريد وتعديله بأمان.
- الهيكلة الجيدة هي ما يجعل كل الخطوات اللاحقة ممكنة: الاختبار والتغليف والأتمتة.

## المختبر خطوة بخطوة

تفضّل أن تقود كل خطوة بنفسك؟ الصق هذه الأوامر واحدًا تلو الآخر بدل الأمر السابق.

### 1. اكشف خطأ الكود القديم

**ما الذي يحدث:** السكربت القديم يعمل دون أي رسالة خطأ، ومع ذلك فنحو 182 من نتائجه خاطئة. ستقارن مخرجاته بالإجابات الموثوقة وتجد السطر الذي يسبب الفرق.

```text
Step 1. Run `python legacy/notebook_v1.py`, then `python scripts/check_golden.py scored_notebook.csv`.
(If scripts/check_golden.py is missing, I will paste it from the lab page.)
About 182 rows should differ from the golden file. Look at the timestamps of the rows that differ,
compare the feature code in legacy/notebook_v1.py with training_features() in
scripts/make_course_assets.py, and show me the exact line that causes the skew.
Explain it in plain words. Do not fix the legacy script.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. طبقة النطاق

**ما الذي يحدث:** تكتب مفردات المشكلة (العملية، الدرجة، القرار) وقاعدة العمل، دون خلطها بأي أدوات تقنية. ودالة واحدة تبني مدخلات النموذج بالطريقة نفسها التي استُخدمت في التدريب.

```text
Step 2. Create src/fraud_service/domain/entities.py and src/fraud_service/domain/policies.py.
If I pasted the code from section 5 of the lab page below, save it exactly. Otherwise write it to this spec:
- entities.py: Channel and Decision (StrEnum), Transaction (pydantic), FeatureVector, RawScore, FraudScore.
  Transaction.to_features() must reproduce training_features() in scripts/make_course_assets.py exactly.
- policies.py: decide(score: RawScore, block_threshold: float) -> Decision, with a review band of 0.15
  below the block threshold.
Only standard-library and pydantic imports. Then walk me through every import and explain why this
layer must not import pandas, joblib or FastAPI.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. طبقة الخدمة

**ما الذي يحدث:** الخدمة تربط الأجزاء ببعضها: تأخذ العملية، وتطلب من النموذج درجة، ثم تطبق قاعدة العمل. وهي تتعامل مع النموذج عبر اتفاق بسيط، فيمكن تركيب أي نموذج آخر لاحقًا.

```text
Step 3. Create src/fraud_service/service/interfaces.py and src/fraud_service/service/scorer.py.
If I pasted the code from the lab page below, save it exactly. Otherwise:
- interfaces.py: a typing.Protocol named Model with an attribute model_version: str and a method
  predict_proba(features: FeatureVector) -> RawScore.
- scorer.py: a dataclass FraudScorer(model: Model, block_threshold: float) with
  score(txn: Transaction) -> FraudScore, which calls txn.to_features(), model.predict_proba() and decide().
Then show me that nothing under service/ imports sklearn, pandas or joblib.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. المحوّل

**ما الذي يحدث:** المحوّل هو المكان الوحيد الذي يعرف scikit-learn وملف النموذج. وإذا تغيرت تقنية النموذج يومًا ما، يتغير هذا الملف وحده.

```text
Step 4. Create src/fraud_service/adapters/sklearn_model.py.
If I pasted the code from the lab page below, save it exactly. Otherwise: a class
SklearnModel(pipeline, model_version) with a classmethod load(path) that reads the joblib bundle
{"pipeline": ..., "version": ...}, and predict_proba(features) that builds a one-row pandas DataFrame from
features.values and returns a RawScore.
It must satisfy the Model protocol without inheriting from it. Explain why that works in Python.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. نقطة الدخول والمقارنة بالملف المرجعي

**ما الذي يحدث:** نقطة الدخول تركّب الأجزاء الحقيقية معًا وتحسب درجات العمليات الخمسة آلاف. ثم تقارن النتائج بالإجابات الموثوقة، ويجب أن يتطابق كل صف.

```text
Step 5. Create src/fraud_service/batch.py, the composition root (or save the version I pasted from the lab page).
It loads SklearnModel from models/fraud_xgb_v3.joblib, builds FraudScorer(block_threshold=0.85),
reads data/transactions_sample.csv and writes scored.csv with the columns
transaction_id, score (9 decimals), decision, model_version. Use argparse with defaults for the options
--input, --output, --model and --threshold.
Then run `python -m fraud_service.batch` and `python scripts/check_golden.py scored.csv`.
The goal is 5,000/5,000 matching rows. If any row differs, find the cause in to_features();
do not touch the golden file.

Reply in Arabic. Keep code, file names and commands in English.
```

### 6. القياس والفحص

**ما الذي يحدث:** فحص سريع للجودة: أداة الفحص تكتشف الأخطاء الصغيرة، ومؤقت الاستيراد يُظهر أن تحميل الكود أصبح شبه فوري، لأن النموذج لا يُحمَّل إلا عند تشغيل البرنامج.

```text
Step 6. Run `ruff check src` and fix what it reports without changing behaviour.
Then run `python -X importtime -c "import fraud_service.service.scorer" 2>&1 | tail -1`
(on Windows PowerShell, run it without the `| tail -1` and read the last line).
Explain the number: why does importing the service no longer load the model or scikit-learn,
and why does that matter for tests and for start-up time?

Reply in Arabic. Keep code, file names and commands in English.
```

### 7. احفظ عملك في Git

**ما الذي يحدث:** تحفظ نسخة من عملك برسالة واضحة، لتعود إلى هذه النقطة متى احتجت.

```text
Step 7. Show me `git status` and summarise what changed in this lab in 3 bullets.
Check that no scored*.csv file and no .venv/ folder will be committed.
Then commit with the message "refactor: extract clean architecture layers from notebook". Do not push.

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 1. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 1 starting point (tag `lab1-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

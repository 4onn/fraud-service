# المختبر 4: بناء مجموعة الاختبارات

المختبر 4 · 50 دقيقة · عمل ثنائي · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m4) · [English version](../lab-4.md)

الهدف: مجموعة اختبارات pytest من ثلاثة مستويات: اختبارات سريعة تنجح في ثوانٍ مع تغطية فروع تتجاوز 80%، ورفض الحمولات المشوّهة الأربعين كلها، واختبارات سلوك تنجح على النموذج الحقيقي.

- **نقطة البداية:** tag `lab4-start`
- **ملفات المختبر:** [`labs/handouts/lab4/`](../handouts/lab4/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-4.md, then teach me Lab 4 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.

Reply in Arabic. Keep code, file names and commands in English.
```

## ما تعلّمه هذه الوحدة بكلمات بسيطة

- قد يخطئ نظام الذكاء الاصطناعي وهو يبدو سليمًا تمامًا، لذا اختبر قراراته لا مجرد أنه يعمل.
- الفحوص الصغيرة السريعة الكثيرة تكشف معظم الأخطاء في ثوانٍ، وقليل من الفحوص البطيئة الدقيقة يحرس الأهم.
- احتفظ بمجموعة إجابات موثوقة وقارن بها عند كل تغيير، لتكتشف أي انحراف صامت مبكرًا.
- فشل الاختبار خبر جيد: فقد اكتشف المشكلة قبل أن يكتشفها العميل.

## المختبر خطوة بخطوة

تفضّل أن تقود كل خطوة بنفسك؟ الصق هذه الأوامر واحدًا تلو الآخر بدل الأمر السابق.

### 1. أدوات الاختبار المشتركة (fixtures)

**ما الذي يحدث:** أدوات مساعدة تشترك فيها الاختبارات: نموذج وهمي يعيد دائمًا درجة معروفة، وعملية نموذجية، وطريقة لتشغيل التطبيق بالنموذج الوهمي بدل الحقيقي.

```text
Step 1. Create tests/conftest.py from the lab (pasted below, if available).
Then explain in one sentence how client_factory gets the fake model into the app (which function
it overrides), and why the real_model fixture is session-scoped.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. اختبارات الوحدة

**ما الذي يحدث:** فحوص صغيرة لقاعدة العمل ولمدخلات النموذج ولحساب الدرجة. وتعمل كلها معًا في جزء من الثانية.

```text
Step 2. Create tests/unit/test_policies.py, tests/unit/test_features.py and tests/unit/test_scorer.py from
the lab (pasted below, if available). Run pytest -m unit. Expected: 16 passed in well under a second.
For each file, tell me in one line which promise of the code it protects.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. اختبارات التكامل

**ما الذي يحدث:** تختبر الخدمة من الخارج، كما سيستخدمها تطبيق العميل: الطلبات الصحيحة تنجح، وكل طلب من الطلبات الأربعين الخاطئة يُرفض.

```text
Step 3. Create tests/integration/test_predict_api.py from the lab (pasted below, if available).
Run pytest -m integration. Expected: 46 passed, 40 of them from one parametrised test over payloads/malformed/.
Explain what parametrisation buys us here.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. اختبارات سلوك النموذج

**ما الذي يحدث:** تفحص هذه الاختبارات قرارات النموذج الحقيقي: لا يجوز أن تتغير الدرجة إذا تغير اسم أو معرّف فقط، ولا أن يبدو المبلغ الأكبر بكثير أكثر أمانًا، ويجب أن تبقى الإجابات الموثوقة الخمسة آلاف كما هي.

```text
Step 4. Create tests/behavioural/test_model_behaviour.py from the lab (pasted below, if available). Run:
pytest -m "behavioural and not slow"     # expected: 5 passed, about 1 second
pytest -m slow                           # expected: the golden-file test, 1 passed, about 20 seconds
Explain in plain words how these tests differ from the unit tests.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. اكسر الكود عمدًا

**ما الذي يحدث:** تعيد الخطأ القديم عمدًا وتراقب أي اختبار يكتشفه. هذا يثبت أن اختباراتك تحميك فعلًا.

```text
Step 5. In src/fraud_service/domain/entities.py, change hour < 6 to hour <= 6.
Run pytest -m "not slow", then pytest -m slow. Tell me which test catches the bug and at which level
of the testing pyramid. Then undo the change and confirm that git diff is empty.

Reply in Arabic. Keep code, file names and commands in English.
```

### 6. نسبة التغطية ثم الحفظ

**ما الذي يحدث:** ترى أي أسطر الكود لا يشغّلها أي اختبار، وتختار أخطرها، ثم تحفظ عملك.

```text
Step 6. Run pytest -m "not slow" --cov --cov-report=term-missing --cov-fail-under=80.
Read the Missing column with me: pick the uncovered line that is riskiest (not the easiest) and
describe the test that would cover it. Record the suite time in BENCHMARKS.md, then commit with
"test: three-level suite with behavioural tripwires".

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 4. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 4 starting point (tag `lab4-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

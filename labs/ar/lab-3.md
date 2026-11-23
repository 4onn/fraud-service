# المختبر 3: وضع خدمة رصد الاحتيال في حاوية

المختبر 3 · 55 دقيقة · في أزواج · تقطعه الاستراحة · [افتح هذا المختبر في موقع الورشة](https://aie113.ibrahimalshehri.sa/ar/#m3) · [English version](../lab-3.md)

الهدف: صورة نحيفة تعمل بمستخدم غير جذري ومزوّدة بفحص سلامة، ومكدس Compose، مع قياساتك الخاصة للحجم وزمن البناء وزمن الاستجابة.

- **نقطة البداية:** tag `lab3-start`
- **ملفات المختبر:** [`labs/handouts/lab3/`](../handouts/lab3/)

## ابدأ مع وكيلك

افتح هذا المجلد في وكيل البرمجة الذكي (Claude Code أو GitHub Copilot في وضع الوكيل أو Cursor أو Codex…) والصق:

```text
Read AGENTS.md and labs/ar/lab-3.md, then teach me Lab 3 one step at a time.
Before each step, explain what we will do and why it matters. Let me run the commands,
compare my output with the expected result, and wait for me to say "next".
At the end, summarise what I learned and ask me 3 short questions.

Reply in Arabic. Keep code, file names and commands in English.
```

## ما تعلّمه هذه الوحدة بكلمات بسيطة

- غلّف الخدمة مع كل ما تحتاجه لتعمل بالطريقة نفسها على أي جهاز، وانسَ عبارة «تعمل على جهازي».
- الحزمة الأصغر والأنظف أسرع في النقل وأكثر أمانًا في التشغيل.
- لا تضع كلمات المرور أو المفاتيح داخل ما تسلّمه، فأي شخص يحصل على الحزمة يستطيع قراءتها.
- شغّل النظام كاملًا على جهازك كما سيعمل في بيئة الإنتاج، قبل أن يعتمد عليه أحد.

## المختبر خطوة بخطوة

تفضّل أن تقود كل خطوة بنفسك؟ الصق هذه الأوامر واحدًا تلو الآخر بدل الأمر السابق.

### 1. ثبّت إصدارات المكتبات

**ما الذي يحدث:** تكتب الإصدار الدقيق لكل مكتبة تحتاجها الخدمة، لنظام Linux داخل الحاوية، حتى يثبّت كل بناء الشيء نفسه تمامًا.

```text
Step 1. Run:
pip install uv
uv pip compile pyproject.toml --python-version 3.12 --python-platform linux --no-annotate --no-header -o requirements.lock
Explain what requirements.lock contains, why it lists packages I never asked for by name, and why we
resolve for Linux even though my laptop may run Windows or macOS.

Reply in Arabic. Keep code, file names and commands in English.
```

### 2. ابنِ صورة بسيطة وازرع سرًا وهميًا

**ما الذي يحدث:** تبني الصورة أولًا بالطريقة التي يتبعها أغلب المبتدئين، وتزرع سرًا وهميًا، لترى المشكلات بعينك.

```text
Step 2. Save Dockerfile.naive from the lab (pasted below, if available). Create the fake secret with
echo "FAKE_API_KEY=sk-not-a-real-key-123" > leaked.env
and build with
time docker build -f Dockerfile.naive -t fraud-service:naive .
(on Windows PowerShell, drop the word time and use Measure-Command if you want the duration).
While it builds, explain what "transferring context" means and why its size matters.

Reply in Arabic. Keep code, file names and commands in English.
```

### 3. افحص ما بنيته

**ما الذي يحدث:** تتحقق من الحجم، وتجد السر الوهمي داخل الصورة، وترى أن الخدمة تعمل بصلاحيات المستخدم الجذر (root) الكاملة.

```text
Step 3. Run:
docker images fraud-service
docker run --rm fraud-service:naive cat leaked.env
docker run --rm fraud-service:naive whoami
Record the image size and the build time in BENCHMARKS.md, and explain the two security problems
these commands prove.

Reply in Arabic. Keep code, file names and commands in English.
```

### 4. ملف الاستثناء وملف Dockerfile الحقيقي

**ما الذي يحدث:** قائمة بالأشياء التي يجب ألا يراها Docker أبدًا، ووصفة من مرحلتين: البناء يتم في غرفة، ولا يُشحن من الأخرى إلا ما يلزم. النتيجة أصغر وأسرع وأكثر أمانًا.

```text
Step 4. Create .dockerignore and the multi-stage Dockerfile from the lab (pasted below, if available).
The Dockerfile installs requirements.lock into /opt/venv in a builder stage; the python:3.12-slim
runtime stage copies only /opt/venv, the model and src/, runs as the non-root user appuser, has a
HEALTHCHECK on /v1/ready and starts uvicorn with the app factory.
Build with:
time docker build -t fraud-service:dev .
Then run docker images fraud-service, whoami and ls /app in the new image, confirm that leaked.env
is not there, and record the new numbers.

Reply in Arabic. Keep code, file names and commands in English.
```

### 5. أثبت أن التخزين المؤقت يعمل

**ما الذي يحدث:** تغيّر سطرًا واحدًا في الكود وتعيد البناء. ولأن الأجزاء الثقيلة لم تتغير، يعيد Docker استخدامها فيستغرق البناء ثواني فقط.

```text
Step 5. Add a comment to src/fraud_service/api/routes.py and rebuild with
time docker build -t fraud-service:dev .
Show me which steps say CACHED, record the rebuild time, and explain why the order of the
instructions in the Dockerfile makes this rebuild take seconds.

Reply in Arabic. Keep code, file names and commands in English.
```

### 6. تشغيل النظام كاملًا بـ Compose

**ما الذي يحدث:** أمر واحد يشغّل النظام كاملًا، أي خدمتك ومعها ذاكرة تخزين مؤقت، كما سيعمل في الإنتاج، مع فحوص السلامة وحد للذاكرة.

```text
Step 6. Create configs/dev.env and docker-compose.yml from the lab (pasted below, if available). Then run:
docker compose up --build -d
docker compose ps        # wait until fraud-api shows (healthy)
and call GET /v1/ready and POST /v1/predict with payloads/sample.json.
Explain what depends_on with condition: service_healthy does, and why configs/dev.env holds no secrets.

Reply in Arabic. Keep code, file names and commands in English.
```

### 7. قِس ونظّف واحفظ

**ما الذي يحدث:** تقيس السرعة والذاكرة داخل الحاوية، وتتأكد أنها تتوقف بهدوء، ثم تحذف الملفات السيئة المتعمدة وتحفظ عملك.

```text
Step 7. Run python scripts/loadtest.py against the container and compare p99 with my Lab 2b number,
run docker stats --no-stream, and time docker compose stop fraud-api. Record all three in BENCHMARKS.md.
Then run docker compose down, delete leaked.env and Dockerfile.naive, show me git status, and commit
with "feat(docker): multi-stage image, healthcheck and compose stack".

Reply in Arabic. Keep code, file names and commands in English.
```

## إذا حدث خطأ

```text
Something went wrong in Lab 3. I ran:
[paste the command]
and got:
[paste the full output or error]
Explain the cause in plain words, then propose the smallest fix.
(If the lab's troubleshooting table lists this symptom, I pasted that row below as well.)
Do not change tests, data or checks to hide the problem.

Reply in Arabic. Keep code, file names and commands in English.
```

## انضممت متأخرًا أو تعثّرت؟

اطلب من وكيلك: "Bring my project to the Lab 3 starting point (tag `lab3-start`), keep my own notes, and explain what changed." الخطوات موجودة في AGENTS.md.

[كل المختبرات](../../README.md)

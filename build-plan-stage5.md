<!-- File: build-plan-stage5.md -->

# سند اجرای مرحله ۵ (Stage 5)
## برنامهٔ ساخت (Build Plan) برای MVP + فاز ۱ خلاقیت + Critique Circles

**تاریخ:** جمعه، ۲۸ نوامبر ۲۰۲۵  
**نسخه:** ۱.۱.۰  
**معماری مرجع:** `architecture-fortified-monolith.md`  
**بریف مرجع:** `technical-brief.md`  
**سند دامنه مرجع:** `product-domain-spec.md`  
**Spec فیچر Critique Circles:** `feature-spec-critique-circles.md`  

این سند، برنامهٔ اجرایی نهایی مرحلۀ ۵ است؛  
از راه‌اندازی Monorepo و زیرساخت تا ساخت MVP، پیاده‌سازی حلقه‌های نقد (Critique Circles) به‌عنوان فیچر تمایزدهندۀ لانچ، و تعبیۀ قلاب‌ها برای هستۀ خلاقیت (Skill Tree، داشبورد شفافیت، Idea Playground با مود Creative Clash).

هیچ تغییری در ساختار کلی و اهداف این سند مجاز نیست، مگر با ADR مستقل.

---

## ۰. پیش‌فرض‌ها و اهداف این مرحله

### ۰.۱. پیش‌فرض‌ها

- **تصمیم معماری:** Fortified Monolith  
  (Monolith ماژولار + Workers ایزوله + Edge هوشمند) – مطابق `architecture-fortified-monolith.md`.
- **تصمیم محتوا:** مدل ContentItem + ContentLinks برای پوشش پیش/پس از چاپ کتاب – مطابق [فصل سوم و بیست‌وششم دامنه](./product-domain-spec.md#chapter-3).
- **تصمیم AI:** بازخورد سقراطی با خروجی JSON ساختاریافته (بدون تصحیح خط‌به‌خط، با ساختار امتیازدهی نسخه‌پذیر) – مطابق [فصل ششم دامنه](./product-domain-spec.md#chapter-6) و سند معماری.  
  سرویس LLM بیرونی (OpenAI/Anthropic) نقش Data Processor دارد و تا حد ممکن با تنظیمات «no-training/no-retention» استفاده می‌شود تا دادهٔ کاربران برای Training عمومی استفاده نشود.
- **تصمیم i18n:** معماری و DB از روز اول دو‌زبانه (fa/en) هستند؛ محتوای EN تدریجی رول‌آوت می‌شود – مطابق [فصل بیست‌ودوم دامنه](./product-domain-spec.md#chapter-22).
- **تصمیم Book Mode:** ساختار فصل‌ها از ابتدا آماده است؛ نمایش UI کتاب/فصل‌ها تا زمان چاپ با Feature Flag خاموش می‌ماند – مطابق [فصل سوم دامنه](./product-domain-spec.md#chapter-3).
- **فیچر تمایزدهندۀ لانچ:** Critique Circles (حلقه‌های نقد همتا) برای تمرین‌های مشخص، با قیود:
  - Exercise-specific (هر حلقه برای یک تمرین مشخص در یک Path مشخص)،
  - Path-specific و Language-specific،
  - Age-band-specific (تفکیک بین حلقه‌های Teen و Adult)،
  - Participation Lock پویا: `N = min(2, تعداد تمرین‌های دیگران در همان حلقه برای آن کاربر)`,
  - مطابق `feature-spec-critique-circles.md` و [فصل هشتم دامنه](./product-domain-spec.md#chapter-8).
- **تصمیم سنی:** حداقل سن ثبت‌نام ۱۳ سال است؛  
  بازه‌های سنی:
  - Teen (۱۳–۱۷)،
  - Adult (۱۸+)،  
  حلقه‌های نقد برای Teen و Adult جداگانه تشکیل می‌شوند؛ زیر ۱۳ سال حساب کاربری ندارند – مطابق [فصل چهاردهم دامنه](./product-domain-spec.md#chapter-14) و `technical-brief.md`.
- **تصمیم حریم خصوصی:** حذف حساب با دو حالت روشن:
  - پیش‌فرض: ناشناس‌سازی مشارکت‌های جامعه‌محور (تمرین‌ها، نقدها، حضور در حلقه‌ها) با ست‌کردن FKهای کاربر به `NULL`؛
  - حالت اختیاری: حذف کامل مشارکت‌ها، در صورت درخواست صریح کاربر – مطابق [فصل هفدهم دامنه](./product-domain-spec.md#chapter-17) و `architecture-fortified-monolith.md`.  
  حذف حساب در MVP به صورت هم‌زمان (synchronous) انجام می‌شود؛ کاربر چند ثانیه منتظر می‌ماند و نتیجه را فوراً می‌بیند.
- **تصمیم مانیتورینگ:** استفاده از استک self-hosted (Prometheus + Grafana + Loki + Sentry) برای Observability – مطابق سند معماری؛ در تمام این سیستم‌ها، فقط IPهای ناشناس‌شده (برای IPv4 و IPv6) ثبت می‌شوند.
- **تصمیم پرداخت:** در MVP:
  - کاربران داخل ایران (country=`IR` در پروفایل): پرداخت تمام محصولات (کتاب چاپی، نسخۀ دیجیتال EPUB، اشتراک‌ها، اعتبار نقد انسانی) از طریق زرین‌پال؛
  - کاربران خارج از ایران (country≠`IR`): پرداخت تمام محصولات (نسخۀ دیجیتال EPUB+NFT، اشتراک‌ها، اعتبار) با USDT روی شبکۀ پالیگان – مطابق [فصل دهم و یازدهم دامنه](./product-domain-spec.md#chapter-10).  
  تشخیص inside/outside به‌صورت صریح بر اساس فیلد `country` پروفایل کاربر انجام می‌شود.
- **تصمیم نسخۀ دیجیتال:**  
  - داخل ایران: فقط EPUB واترمارک‌شده (بدون NFT)؛  
  - خارج ایران: EPUB واترمارک‌شده + NFT روی پالیگان؛  
  - پس از تحویل دیجیتال (دانلود/ایجاد NFT)، بازپرداخت وجود ندارد – مطابق [فصل دوازدهم و سیزدهم دامنه](./product-domain-spec.md#chapter-12).
- **تصمیم لایک و Analytics:**  
  - دادهٔ عملیاتی لازم برای کارکرد لایک (بر پایهٔ device fingerprint) همیشه ذخیره می‌شود و به user_id متصل نیست؛  
  - ثبت رویداد تحلیلی لایک در `events_behavior` فقط با Opt-in کاربر فعال می‌شود – مطابق [فصل چهارم و هفدهم دامنه](./product-domain-spec.md#chapter-4).
- **تصمیم URL کدهای پاسخ‌گو:**  
  - ساختار `/[lang]/book/v{n}/chapter/{id}/` (مثلاً `/fa/book/v1/chapter/3/`) به‌عنوان مبنای طراحی – مطابق [فصل سوم دامنه](./product-domain-spec.md#chapter-3).
- **تصمیم فرم ثبت‌نام:**  
  - فیلدهای ضروری: `email`, `display_name`, `date_of_birth`, `country`, `preferred_language`, `gender`؛  
  - فیلدهای اختیاری: `first_name`, `last_name`, `phone`, `city`;  
  - نام مستعار (display_name) ۳–۱۵ کاراکتر با قوانین ممنوعیت محتوای توهین‌آمیز/جعل هویت.
- **تصمیم Auth داخلی:**  
  - در جدول `users` یک فیلد `auth_user_id` به صورت UUID، یونیک و non-null نگه‌داری می‌شود تا پیوند ۱:۱ با User در Supabase Auth برقرار باشد.
- **تصمیم بازی خلاقیت:** Idea Playground با مود Creative Clash به‌عنوان فیچر خلاقیت MVP پیاده‌سازی می‌شود؛ در MVP، خروجی متنی Creative Clash به‌صورت پیش‌فرض در پایگاه‌داده ذخیره نمی‌شود (Persist جرقه‌ها برای فاز بعدی رزرو است)، و فقط آمار تجمیعی سطح بالا ممکن است در Analytics ثبت شود. مودهای دیگر (مینی‌داستان، اتاق فرار کلمات) برای فاز بعد رزرو هستند – مطابق [فصل دوم و پنجم دامنه](./product-domain-spec.md#chapter-2), `technical-brief.md`.

### ۰.۲. اهداف مرحله ۵

- آماده‌سازی کامل Monorepo و محیط توسعه.
- تعریف و اعمال اسکیما در Postgres، شامل:
  - فیلدهای هویتی کاربران (نام مستعار، تاریخ تولد، کشور، زبان، جنسیت، شهر، تلفن) و فیلد `auth_user_id` برای پیوند با Supabase؛
  - `language` روی جداول محتوایی/آموزشی؛
  - مدل Critique Circles با `path_id`, `exercise_id`, `language`, `age_band` و رفتار ON DELETE مطابق Spec؛
  - ساختار `ai_feedback.payload` به صورت JSONB و score map؛
  - جدول reactions برای دادهٔ عملیاتی لایک‌ها (device-based).
- پیاده‌سازی Core Backend (ماژول‌های دامنه‌ای).
- پیاده‌سازی Worker AI با Prompt سقراطی و خروجی JSON.
- پیاده‌سازی کامل Critique Circles در سطح MVP، مطابق `feature-spec-critique-circles.md` (شامل Participation Lock پویا).
- پیاده‌سازی Next.js App (Public + App + Admin) با i18n، صفحات Critique و صفحهٔ Idea Playground (Creative Clash).
- راه‌اندازی Edge (Cloudflare)، Bot دیسکورد، مانیتورینگ، تست کامل و استقرار MVP.
- پیاده‌سازی خط‌لولهٔ کتاب دیجیتال: واترمارک EPUB + DownloadToken + Mint NFT (برای کاربران خارج) به‌صورت خودکار.
- تضمین انطباق پیاده‌سازی با سیاست‌های داده/حریم خصوصی/سنی و مدل پرداخت داخلی/خارجی.

---

## فاز ۰: زیرساخت و پایه‌ریزی (Infrastructure & Foundation)

### ۰.۱. Monorepo و ابزارها

1. **ایجاد Monorepo با Turborepo و pnpm:**
   - ساختار ریشه:
     - `apps/web` – اپ Next.js (Public + App + Admin + Playground).
     - `apps/api` – Backend Monolith.
     - `apps/worker-ai` – Worker AI Feedback.
     - `apps/worker-crypto` – Worker Crypto/NFT (در MVP فعال برای پرداخت‌های خارج).
     - `apps/worker-publisher` – Worker نشر/واترمارک EPUB و صدور DownloadToken.
     - `packages/database` – اسکیما و DB Client (Prisma/Drizzle).
     - `packages/shared` – Types مشترک، DTOها، Zod Schemas، utilهای دامنه‌ای.
   - فایل‌های ریشه:
     - `package.json` – اسکریپت‌های root (`dev`, `build`, `lint`, `test`).
     - `pnpm-workspace.yaml` – تعریف workspaceها (`apps/*`, `packages/*`).
     - `turbo.json` – تعریف pipelineهای Turborepo.
     - `tsconfig.base.json` – تنظیمات TypeScript پایه (strict + paths).

2. **پیکربندی TypeScript Strict:**
   - مطابق سند معماری، `strict: true`, `noImplicitAny: true`, `strictNullChecks: true`.
   - تعریف aliasها:
     - `@shared/*` → `packages/shared/src/*`
     - `@db/*` → `packages/database/src/*`
     - `@api/*` → `apps/api/src/*`
     - `@web/*` → `apps/web/src/*`

3. **ابزارهای کیفیت کد:**
   - `.eslintrc.cjs` با پیکربندی مناسب TS/React/Node.
   - `.prettierrc.cjs` برای فرمت یکپارچه.
   - اسکریپت‌های `"lint"`, `"format"`, `"test"` در root و sub-packages.

### ۰.۲. محیط توسعه (Docker Compose)

4. **تنظیم `docker/docker-compose.dev.yml`:**
   - سرویس‌ها:
     - PostgreSQL 18.1 (پورت ۵۴۳۲).
     - Valkey 9.0.0 (پورت ۶۳۷۹).
     - Meilisearch 1.27.0 (پورت ۷۷۰۰).
     - MinIO به‌عنوان S3 شبیه‌ساز (پورت‌های استاندارد).
   - تعریف Volumes برای نگهداشت داده‌های محلی.

5. **(اختیاری اما توصیه‌شده) Compose مانیتورینگ محلی:**
   - Prometheus، Grafana، Loki در یک `docker-compose.monitoring.yml` مجزا برای تست محلی استک Observability.

### ۰.۳. i18n و Feature Flags

6. **پیکربندی i18n در `apps/web/next.config.mjs`:**
   - `locales: ['fa', 'en']`, `defaultLocale: 'fa'`.

7. **ایجاد ساختار فایل‌های ترجمه:**
   ```text
   apps/web/locales/fa/common.json
   apps/web/locales/fa/dashboard.json
   apps/web/locales/fa/auth.json
   apps/web/locales/fa/circles.json
   apps/web/locales/fa/playground.json
   ...
   apps/web/locales/en/common.json
   apps/web/locales/en/dashboard.json
   apps/web/locales/en/auth.json
   apps/web/locales/en/circles.json
   apps/web/locales/en/playground.json
   ...
   ```

8. **تعریف Feature Flags در env:**
   - در `.env.example`:
     - `NEXT_PUBLIC_ENABLE_BOOK_MODE=false`
     - `NEXT_PUBLIC_API_BASE_URL=...`
     - `CRITIQUE_REQUIRED_REVIEWS=2` (مقدار سقف N؛ در منطق N=min(2, #others) استفاده می‌شود)
     - سایر متغیرها (Supabase, Sentry, …).

### ۰.۴. Observability و CI/CD

9. **راه‌اندازی استک مانیتورینگ:**
   - Prometheus:
     - Scrape کردن `/metrics` از `apps/api` و `apps/worker-*`.
   - Grafana:
     - داشبوردهای پایه:
       - Latency و Error Rate APIها (TTFB، SLA Critique APIs).
       - Backlog صف BullMQ (AI Worker، Publishing Worker، Crypto Worker).
   - Loki:
     - تجمیع لاگ‌ها از سرویس‌ها.
   - Sentry:
     - برای Frontend و Backend؛  
       لاگ‌ها و رویدادها فقط از IP ناشناس‌شده (از هدر `X-Anonymized-IP`) استفاده می‌کنند.

10. **راه‌اندازی CI (مثلاً GitHub Actions):**
    - Workflow `ci.yml`:
      - `pnpm install`
      - `pnpm lint`
      - `pnpm test`
      - `pnpm build`
    - در مراحل بعد، اتصال به فرایند Deploy (Hetzner/AWS/لیارا/ابرآروان).

---

## فاز ۱: مدل داده و لایهٔ پایگاه‌داده (Core Data Layer)

### ۱.۱. پیاده‌سازی اسکیما در `packages/database`

11. **تعریف مدل‌ها در Prisma/Drizzle (`schema.prisma` یا معادل):**

- **کاربران و نقش‌ها:**
  - `User`:
    - `id`: uuidv7
    - `auth_user_id`: UUID, UNIQUE, NOT NULL  ← پیوند ۱:۱ با کاربر Supabase
    - `email`: TEXT, UNIQUE, NOT NULL
    - `display_name`: TEXT, NOT NULL (Validation: ۳–۱۵ کاراکتر)
    - `date_of_birth`: DATE, NOT NULL
    - `country`: TEXT, NOT NULL
    - `preferred_language`: TEXT, NOT NULL (`fa` | `en`)
    - `gender`: TEXT, NOT NULL (`female` | `male` | `non_binary` | `prefer_not_to_say`)
    - `city`: TEXT, NULL
    - `phone`: TEXT, NULL
    - `created_at`, `updated_at`
  - `Role`, `UserRole`.

- **فصل‌ها و نسخه‌ها:**
  - `Chapter`, `ChapterVersion` با `language`, `bookVersion`, `title`, `body`, `status`, `isCurrent`.

- **محتوا:**
  - `ContentItem`, `ContentLink`.

- **مسیرها و تمرین‌ها:**
  - `LearningPath`, `Session`, `Exercise` (`language`, `isCritiqueEnabled`).
  - `Submission` با:
    - `user_id`,
    - `exercise_id`,
    - `language`,
    - `draft_content`, `final_content`,
    - `status`,
    - `submitted_at`,
    - `circle_id?` (FK به `CritiqueCircle` با `ON DELETE SET NULL`),
    - `peer_feedback_unlocked` (bool).

- **بازخورد:**
  - `AiFeedback` با `payload JSONB`.
  - `HumanFeedback`.

- **Critique Circles (مطابق `feature-spec-critique-circles.md`):**
  - `CritiqueCircle`:
    - `id`, `name?`,  
    - `status` (`open` | `active` | `closed`),
    - `capacity` (default: 3، بین ۲ تا ۱۰),
    - `language`,
    - `path_id` (NOT NULL),
    - `exercise_id` (NOT NULL),
    - `age_band` (`teen` | `adult`),
    - `owner_id?` با `ON DELETE RESTRICT`,
    - `created_at`, `updated_at`.
  - `CircleMember`:
    - `circle_id` → `CritiqueCircle(id)` با `ON DELETE CASCADE`,
    - `user_id` → `User(id)` با `ON DELETE RESTRICT`,
    - `role` (`member` | `owner`),
    - `joined_at`, `left_at?`,
    - `UNIQUE(circle_id, user_id)`.
  - `CircleSubmission`:
    - `circle_id` → `CritiqueCircle(id)` با `ON DELETE CASCADE`,
    - `submission_id` → `Submission(id)` با `ON DELETE CASCADE`,
    - `author_id` → `User(id)` با `ON DELETE RESTRICT`,
    - `UNIQUE(circle_id, submission_id)`.
  - `PeerFeedback`:
    - `circle_id` → `CritiqueCircle(id)` با `ON DELETE CASCADE`,
    - `submission_id` → `Submission(id)` با `ON DELETE CASCADE`,
    - `reviewer_id` → `User(id)` با `ON DELETE RESTRICT`,
    - `body`, `score?`, `created_at`,
    - `UNIQUE(circle_id, submission_id, reviewer_id)`.

- **Featured & Consents:**
  - `FeaturedCandidate`, `FeaturedSample`, `Consent`.

- **Commerce:**
  - `Transaction`, `Subscription`, `UserCredit`,
  - `CryptoPayment`, `WalletTokenLink`,
  - `DownloadToken`.

- **Compliance & Analytics:**
  - `Report`, `DeletionRequest`, `SecurityEvent`,
  - `EventBehavior`, `EventTechnical`.

- **Reactions (لایک‌ها):**
  - `Reaction`:
    - `id`: UUID PRIMARY KEY,
    - `device_fingerprint`: TEXT NOT NULL,
    - `content_id`: UUID NOT NULL,
    - `reaction_type`: TEXT NOT NULL (مثلاً `'like'`),
    - `created_at`, `updated_at`,
    - `UNIQUE (device_fingerprint, content_id, reaction_type)`؛  
    این جدول به user_id متصل نیست و اطلاعات عملیاتی لایک را ذخیره می‌کند.

12. **افزودن ستون `language TEXT NOT NULL`** به تمام جداول محتوایی/آموزشی لازم (LearningPath, ChapterVersion, ContentItem, Exercise, Submission).

13. **تعریف ستون‌های سنی برای Critique Circles:**
    - `age_band TEXT NOT NULL CHECK (age_band IN ('teen','adult'))` در `critique_circles`.

14. **تولید و اعمال Migrationها:**
    - اجرای migration روی DB توسعه.
    - بررسی صحت FKها، خصوصاً:
      - `ON DELETE RESTRICT` برای FKهای User در Critique و Community،
      - `ON DELETE SET NULL` برای `submissions.circle_id`.

15. **تعریف ایندکس‌های کلیدی:**
   - برای کارایی Critique و Analytics:
     - `submissions(user_id, exercise_id)`
     - `submissions(circle_id)`
     - `circle_members(circle_id, user_id)`
     - `circle_submissions(circle_id)`
     - `peer_feedback(circle_id, reviewer_id)`
     - `peer_feedback(submission_id)`
     - `critique_circles(path_id, exercise_id, language, age_band)`
     - `events_behavior(user_id, event_type, created_at)`
     - `reactions(device_fingerprint, content_id, reaction_type)`
     - `security_events(user_id, created_at)` (در صورت وجود).

16. **Seed داده‌های تست Critique Circles:**
   - ایجاد:
     - یک حلقۀ `open` (teen) با ۲ عضو؛
     - یک حلقۀ `active` (adult) با ۳ عضو و چند نقد؛
     - Submissionهایی با `peer_feedback_unlocked=true/false`.
   - این Seed برای تست‌های Integration در فاز ۵ استفاده می‌شود.

---

## فاز ۲: Core Backend (Monolith API)

### ۲.۱. Identity & Access Module (`apps/api/src/modules/identity`)

17. **ادغام با Supabase Auth:**
   - استفاده از `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`.
   - Middleware `auth` برای استخراج User از Session/JWT و sync با `users` از روی `auth_user_id`.

18. **ایجاد/به‌روزرسانی پروفایل کاربر:**
   - در اولین ثبت‌نام/ورود:
     - دریافت فیلدهای ثبت‌نام (ایمیل، نام مستعار، تاریخ تولد، کشور، زبان، جنسیت) از Frontend؛
     - Validation نام مستعار (طول، کاراکترها، کلمات ممنوع)؛
     - محاسبۀ سن از `date_of_birth`:
       - اگر `age < 13` → خطا (عدم اجازهٔ ثبت‌نام)؛
       - تنظیم فیلدهای User با گروه سنی مناسب (Teen/Adult در منطق اپ، اگر لازم باشد).

### ۲.۲. Learning & Practice Module (`modules/learning`)

19. **Endpoints مسیرها و تمرین‌ها:**
   - `GET /paths`, `GET /paths/{id}`, `GET /exercises/{id}`.

20. **Submission Flow:**
   - `POST /paths/{pathId}/exercises/{exerciseId}/start` → ایجاد Submission (`draft`).
   - `PATCH /submissions/{id}` → Autosave.
   - `POST /submissions/{id}/submit`:
     - Validate:
       - مالکیت Submission،
       - حداقل طول متن،
       - محدودیت تعداد ارسال در روز:
         - مقدار پیش‌فرض: **۱۰ Submit نهایی در ۲۴ ساعت**؛
         - شمارش بر اساس تغییر `status` از `draft` به `submitted` انجام می‌شود؛
         - حذف بعدی Submission، سهمیه را آزاد نمی‌کند؛
         - مقدار در Config (مثلاً `MAX_SUBMISSIONS_PER_DAY`) پیاده‌سازی شده و قابل تغییر از طریق ADR است.
     - `status='submitted'`, `submitted_at=now()`,
     - enqueue به `feedback:ai` (در صورت فعال بودن AI برای آن تمرین).

---

### ۲.۳. Feedback Module (`modules/feedback`)

21. **AI Feedback:**
   - Service `enqueueAiFeedback(submissionId)` → گذاشتن Job در `feedback:ai`.
   - `GET /submissions/{id}/feedback` → ترکیب AI + Human Feedback.

22. **Human Feedback برای منتورها:**
   - `GET /mentor/submissions` → صف مشترک Submissionهای نیازمند نقد انسانی.
   - `POST /mentor/submissions/{id}/feedback` → ثبت نقد.

---

### ۲.۴. Content Module (`modules/content`)

23. **ContentItem CRUD و Links:**
   - `GET /content-items`, `GET /content-items/{id}`،
   - Admin: `POST/PATCH/DELETE /content-items`, `POST /content-links`.

---

### ۲.۵. Community Module (`modules/community`)

این ماژول، پیاده‌سازی Critique Circles و Featured Samples را طبق `feature-spec-critique-circles.md` و [فصل هشتم/نهم دامنه](./product-domain-spec.md#chapter-8) انجام می‌دهد.

24. **Critique Circles – APIها:**
   - `POST /api/circles/join`
   - `GET /api/circles/my`
   - `POST /api/peer-feedback`
   - `POST /api/circles/report`

   منطق دقیق مطابق بخش ۴.۴ همین سند و `feature-spec-critique-circles.md` است، شامل:
   - استفاده از `(exercise_id, path_id, language, age_band)` برای یافتن/ایجاد حلقهٔ `open`,
   - محدودیت حلقه برای کاربران عادی/دارای اشتراک فعال بر اساس تعریف «حلقۀ فعال برای کاربر»،
   - Participation Lock پویا: محاسبۀ N به‌صورت `N = min(2, تعداد Submissionهای دیگران در همان حلقه برای کاربر)`,
   - جداسازی حلقه‌های Teen و Adult.

25. **Featured Samples:**
   - `GET /featured-samples` (Public).
   - Admin routes برای انتخاب/ویرایش/منتشر کردن نمونه‌ها.

---

### ۲.۶. Commerce Module (`modules/commerce`)

26. **پرداخت زرین‌پال (داخل ایران):**
   - `POST /payments/zarinpal/intents` → Intent + لینک درگاه.
   - `GET /payments/zarinpal/callback` → Verify + ثبت Transaction +:
     - فعال‌سازی `subscriptions` (در صورت خرید اشتراک)،
     - افزایش `user_credits` (در صورت خرید اعتبار نقد انسانی)،
     - ثبت خرید EPUB و راه‌اندازی Job واترمارک/DownloadToken (در صورت خرید کتاب دیجیتال).

27. **پرداخت کریپتو (خارج از ایران):**
   - مقداردهی بر اساس فیلد `country` کاربر:
     - اگر `country != 'IR'`، گزینهٔ پرداخت با کریپتو فعال است.
   - `POST /payments/crypto/intents` → ثبت Intent برای USDT/Polygon (مبلغ، محصول، userId).
   - Worker Crypto:
     - Poll/Listen شبکه برای txHash؛
     - Verify مبلغ و مقصد؛
     - ثبت در `CryptoPayment`؛
     - فعال‌سازی محصول:
       - برای کتاب دیجیتال: راه‌اندازی Publishing Worker جهت واترمارک EPUB و صدور DownloadToken + Mint NFT؛
       - برای اشتراک/اعتبار: فعال‌سازی رکوردهای `Subscription`/`UserCredit`.
   - در موارد خطای خودکار، پشتیبانی می‌تواند txHash را که کاربر از طریق ایمیل/دیسکورد فرستاده است، به‌صورت دستی در سیستم تأیید کند؛ اما مسیر اصلی همان Intent+API+Worker است.

---

### ۲.۷. Compliance & Analytics Modules

28. **حذف حساب (`modules/compliance`):**
   - `POST /me/delete-account`:
     - Frontend دو گزینه را نمایش می‌دهد (ناشناس‌سازی/حذف کامل)،
     - Backend Request را در `deletion_requests` ثبت کرده و `AccountDeletionService` را به‌صورت هم‌زمان اجرا می‌کند.

29. **گزارش تخلف (`modules/compliance`):**
   - `POST /reports` – فرم عمومی گزارش.
   - `GET /admin/reports` – برای ناظر حقوقی، با فیلتر بر حسب context (از جمله `critique_circle_peer_feedback`).

30. **Analytics (`modules/analytics`):**
   - Endpoint برای جمع‌آوری `events_behavior`, `events_technical` (Opt-in برای کاربران لاگین‌شده؛ برای مهمان‌ها بدون user_id).
   - Jobهای Retention (حذف داده‌های قدیمی طبق سیاست دامنه).

---

## فاز ۳: Workers (AI, Publishing, Crypto)

### ۳.۱. AI Worker (`apps/worker-ai`)

31. **اتصال به Queue و DB:**
   - راه‌اندازی BullMQ روی Valkey.
   - استفاده از `packages/database/client`.

32. **Job `feedback:ai`:**
   - ورودی: `submissionId`.
   - مراحل:
     1. دریافت Submission و زبان؛
     2. ساخت Prompt سقراطی؛
     3. فراخوانی LLM (با تنظیمات no-training/no-retention در صورت امکان)؛
     4. Validate ساختار JSON مطابق Schema؛
     5. ذخیره در `AiFeedback` و update `submissions.status`.

33. **مدیریت خطا و Retry:**
   - Retry ۳ بار با Backoff؛
   - ثبت خطا در Sentry.

### ۳.۲. Publishing Worker (`apps/worker-publisher`)

34. **Watermark EPUB + DownloadToken:**
   - ورودی Job: `ebookId`, `userId`/`userEmail`.
   - گرفتن EPUB خام، واترمارک با هش ایمیل، ذخیرهٔ فایل واترمارک‌شده، ایجاد DownloadToken.

35. **اسکلت Anthology Job:**
   - تعریف Queue `publisher:anthology`، فعلاً فقط logging (پیاده‌سازی کامل در فاز بعد).

### ۳.۳. Crypto Worker (`apps/worker-crypto`)

36. **Worker Crypto/NFT:**
   - Job `crypto:verify` با `txHash`, `userId`, `productType`, `productId`.
   - در MVP:
     - Verify مبلغ و مقصد تراکنش USDT/Polygon؛
     - ثبت در `crypto_payments`;
     - فعال‌سازی محصول (اشتراک، اعتبار، یا Ebook);
     - در صورت Ebook:
       - فراخوانی Publishing Worker جهت واترمارک EPUB و صدور DownloadToken؛
       - Mint NFT و ثبت در `wallet_token_links`.

---

## فاز ۴: Frontend (Next.js App)

### ۴.۱. صفحات عمومی (Public)

37. **صفحهٔ اصلی (`/[lang]`):**
   - معرفی پروژه،
   - نمایش بخشی از Skill Tree (آمار جمعی: تعداد تمرین‌ها، مسیرها، کاربران فعال).

38. **Learning Hub (`/[lang]/resources`):**
   - فهرست ContentItemها با فیلتر/جست‌وجو.

39. **Idea Playground (`/[lang]/playground`):**
   - مود Creative Clash:
     - سه گردونۀ تصادفی برای شخصیت/مکان/شیء،
     - تایمر ۶۰ ثانیه‌ای،
     - فیلد متنی برای Logline،
     - دکمهٔ «ثبت» و پیام تشویقی (بدون نمره).
   - در MVP:
     - خروجی متنی Logline در DB Persist نمی‌شود؛
     - تنها شمارش‌های کلی استفاده (در سطح رویدادهای رفتاری غیرشخصی) ممکن است برای تحلیل UX ثبت شود.

40. **صفحات کتاب (`/[lang]/book/v1/chapter/[id]`):**
   - UI فصل‌ها؛
   - فقط وقتی `NEXT_PUBLIC_ENABLE_BOOK_MODE=true` است فعال؛ در غیر این‌صورت پیام انتشار کتاب.

### ۴.۲. App کاربری

41. **Auth (`/[lang]/login`, `/[lang]/register`):**
   - اتصال به Supabase،
   - فرم ثبت‌نام شامل:
     - ایمیل،
     - نام مستعار (با Validation طول/کاراکتر/محتوا)،
     - رمز عبور،
     - تاریخ تولد،
     - کشور،
     - زبان ترجیحی،
     - جنسیت (با گزینهٔ «مایل به گفتن نیستم»),
     - فیلدهای اختیاری: نام/نام خانوادگی، شماره تلفن، شهر.
   - هندل خطاها (مثلاً سن < ۱۳، نام مستعار نامعتبر، Email تکراری).
   - در ورود با Discord، پس از بازگشت از OAuth، کاربر در صورت نداشتن پروفایل کامل به صفحهٔ تکمیل پروفایل هدایت می‌شود و بدون تکمیل این فرم اجازهٔ ورود به مسیرها و پرداخت را ندارد.

42. **داشبورد (`/[lang]/dashboard`):**
   - مسیرها، لیست تمرین‌های در حال انجام و تکمیل‌شده، خلاصه رشد فردی.
   - لینک به Idea Playground برای warm-up.

43. **صفحهٔ تمرین (`/[lang]/exercise/[id]`):**
   - توضیح تمرین، ContentLinks، ادیتور متن، Autosave، Submit.
   - در صورت فعال بودن Critique، پس از Submit پیشنهاد پیوستن به حلقه.

44. **صفحهٔ بازخورد تمرین:**
   - نمایش AI Feedback + Human Feedback.

45. **صفحهٔ حلقه‌های نقد (`/[lang]/circles`):**
   - نمایش حلقۀ مرتبط با کاربر (بر اساس منطق `GET /api/circles/my`):
     - اعضا، تمرین‌های دیگران، وضعیت نقدهای user،
     - تمرین خود کاربر و وضعیت `peer_feedback_unlocked`,
   - نمایش هشدار بنری برای:
     - کاربران عادی که در یک حلقۀ فعال هستند،
     - کاربران دارای اشتراک فعال که در ≥۳ حلقۀ فعال هستند.

46. **صفحات تنظیمات و شفافیت:**
   - `/[lang]/me/settings`: مدیریت نام نمایشی، زبان، Opt-in Analytics، حذف حساب (با دو گزینه) و اطلاع صریح دربارۀ قطع دسترسی آنلاین مجدد به کتاب دیجیتال پس از حذف حساب.
   - `/[lang]/me/transparency`: خلاصهٔ داده‌ها، روند scoreهای AI، وضعیت درخواست‌های حذف/داده.

### ۴.۳. Admin UI

47. **پنل مدیر محتوا (`/[lang]/admin/content`):**
   - مدیریت فصل‌ها، ContentItemها، Links، فعال/غیرفعال کردن Critique برای تمرین‌ها.

48. **پنل منتور (`/[lang]/admin/mentors`):**
   - صف Submissionهای نیازمند نقد انسانی، Claim/Release.

49. **پنل حقوقی (`/[lang]/admin/reports`):**
   - مشاهده و فیلتر گزارش‌های تخلف، از جمله گزارش‌های Critique Circles با context خاص.

---

## فاز ۵: Edge، Discord، تست و استقرار

### ۵.۱. Cloudflare

50. **DNS و CDN:**
   - تنظیم DNS دامنهٔ اصلی،
   - فعال‌سازی CDN برای HTML/استاتیک‌ها.

51. **IP Anonymizer Worker:**
   - پیاده‌سازی Worker مطابق سند معماری (خواندن `CF-Connecting-IP`، ماسک IPv4 به شکل `x.y.z.0` و IPv6 به شکل نگه‌داشتن پیشوند /64، ارسال در `X-Anonymized-IP`).

52. **WAF و Rate Limiting:**
   - پیکربندی WAF برای محافظت در برابر XSS, SQLi, …،
   - Rate Limits برای Auth, Submit, Feedback, Reports و Critique APIs و واکنش‌ها.

### ۵.۲. Discord Bot

53. **Bot Setup:**
   - ساخت Bot، دریافت Token، اتصال به سرور.

54. **ویژگی‌های MVP:**
   - Sync نقش‌های `user`/`mentor`/`premium`،
   - اعلان تمرین جدید، آماده شدن بازخورد مهم،
   - رعایت محدودیت سنی در دیسکورد:
     - کاربران ۱۳–۱۵: نقش read-only روی کانال‌های مشخص،
     - کاربران ۱۶–۱۷ و ۱۸+: امکان ارسال پیام طبق قوانین عمومی.

### ۵.۳. تست و استقرار

55. **تست‌های واحد و Integration (مسیر اصلی کاربر):**
   - ثبت‌نام (با فیلدهای هویتی) → ورود → انتخاب مسیر → start/submit تمرین → دریافت AI Feedback → پیوستن به حلقۀ نقد → نوشتن نقد همتا → unlock شدن نقدهای دریافتی.

56. **سناریوهای تست محدودیت حلقه‌ها (طبق توافق):**

| سناریو | کاربر | گروه سنی | عمل | نتیجهٔ مورد انتظار |
|:-------|:------|:--------|:----|:-------------------|
| ۱ | عادی | Adult | join به حلقهٔ اول (exercise X, path A) | ✅ موفق |
| ۲ | عادی | Adult | join به حلقهٔ دوم در حالی که حلقۀ اول هنوز برای او فعال است (`peer_feedback_unlocked=false`) | ❌ خطای `CIRCLE_LIMIT_REACHED` |
| ۳ | دارای اشتراک فعال | Adult | join در Path A | ✅ موفق |
| ۴ | دارای اشتراک فعال | Adult | join در Path B | ✅ موفق |
| ۵ | دارای اشتراک فعال | Adult | join دوم در همان Path A | ❌ خطا (`CIRCLE_LIMIT_REACHED_FOR_PATH`) |
| ۶ | دارای اشتراک فعال | Adult | join در Path C (سومین حلقه، در حالی که دو حلقۀ قبلی برای او فعال‌اند) | ✅ موفق + `warning=true` |
| ۷ | دارای اشتراک فعال با ۳ حلقه فعال | Adult | مشاهدهٔ `/circles` | نمایش هشدار بنری |
| ۸ | عادی | Teen | join به حلقهٔ Teen | ✅ موفق، حلقه با `age_band='teen'` ساخته می‌شود |
| ۹ | عادی | Teen | تلاش برای join به حلقه‌ای که `age_band='adult'` دارد | ❌ جلوگیری (حلقۀ Adult برای Teen ایجاد/استفاده نمی‌شود) |
| ۱۰ | عادی | Adult | پیوستن به حلقۀ دونفره (ظرفیت ۲، فقط دو عضو) و نوشتن یک نقد روی تمرین دیگر | ✅ پس از یک نقد، `peer_feedback_unlocked=true` برای او (N=min(2,1)=1) |

57. **سایر تست‌های Integration:**
   - BullMQ (Retry, Failure Handling),
   - Meilisearch (جست‌وجوی FA/EN),
   - Worker Publishing (صدور Watemark EPUB و DownloadToken),
   - Crypto Worker (پایش پرداخت USDT، Mint NFT),
   - AccountDeletionService (دو حالت حذف حساب و تأثیر روی Critique/Digital Access، اجرای synchronous),
   - Idea Playground – Creative Clash (تایمر، کارکرد بدون Auth و با Auth، عدم Persist محتوایی).

58. **استقرار (Deployment):**
   - ساخت Docker imageها برای:
     - `apps/web`, `apps/api`, `apps/worker-ai`, `apps/worker-publisher`, `apps/worker-crypto`.
   - استقرار روی زیرساخت انتخاب‌شده (Hetzner/AWS/لیارا/ابرآروان)،
   - استفاده از Staging برای تست نهایی،
   - در صورت امکان Rollout امن (Blue/Green/Canary).

---

این برنامهٔ ساخت، اجرای کامل MVP + فاز ۱ خلاقیت (Skill Tree + داشبورد شفافیت + Idea Playground/Creative Clash) + Critique Circles را  
بر اساس معماری Fortified Monolith، تصمیم‌های دامنه‌ای (۳۳ فصل)، و مشخصات فنی تفصیلی تضمین می‌کند.  
هر قدم توسعه باید مطابق این فازها و تسک‌ها انجام شود، و هر تغییری در دامنه/معماری نیازمند ADR مستقل است.
```

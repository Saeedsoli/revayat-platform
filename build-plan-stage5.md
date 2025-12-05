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
- **تصمیم i18n:** معماری و DB از روز اول دو‌زبانه (fa/en) هستند؛ محتوای EN تدریجی رول‌آوت می‌شود – مطابق [فصل بیست‌ودوم دامنه](./product-domain-spec.md#chapter-22).
- **تصمیم Book Mode:** ساختار فصل‌ها از ابتدا آماده است؛ نمایش UI کتاب/فصل‌ها تا زمان چاپ با Feature Flag خاموش می‌ماند – مطابق [فصل سوم دامنه](./product-domain-spec.md#chapter-3).
- **فیچر تمایزدهندۀ لانچ:** Critique Circles (حلقه‌های نقد همتا) برای تمرین‌های مشخص، با قیود:
  - Exercise-specific (هر حلقه برای یک تمرین مشخص در یک Path مشخص)،
  - Path-specific و Language-specific،
  - Age-band-specific (تفکیک بین حلقه‌های Teen و Adult)،
  - مطابق `feature-spec-critique-circles.md` و [فصل هشتم دامنه](./product-domain-spec.md#chapter-8).
- **تصمیم سنی:** حداقل سن ثبت‌نام ۱۳ سال است؛  
  بازه‌های سنی:
  - Teen (۱۳–۱۷)،
  - Adult (۱۸+)،  
  حلقه‌های نقد برای Teen و Adult جداگانه تشکیل می‌شوند؛ زیر ۱۳ سال حساب کاربری ندارند – مطابق [فصل چهاردهم دامنه](./product-domain-spec.md#chapter-14) و `technical-brief.md`.
- **تصمیم حریم خصوصی:** حذف حساب با دو حالت روشن:
  - پیش‌فرض: ناشناس‌سازی مشارکت‌های جامعه‌محور (تمرین‌ها، نقدها، حضور در حلقه‌ها) با ست‌کردن FKهای کاربر به `NULL`؛
  - حالت اختیاری: حذف کامل مشارکت‌ها، در صورت درخواست صریح کاربر – مطابق [فصل هفدهم دامنه](./product-domain-spec.md#chapter-17) و `architecture-fortified-monolith.md`.
- **تصمیم مانیتورینگ:** استفاده از استک self-hosted (Prometheus + Grafana + Loki + Sentry) برای Observability – مطابق سند معماری.
- **تصمیم پرداخت:** در MVP:
  - کاربران داخل ایران: پرداخت تمام محصولات (کتاب چاپی، نسخۀ دیجیتال EPUB، اشتراک‌ها، اعتبار نقد انسانی) از طریق زرین‌پال؛
  - کاربران خارج از ایران: پرداخت تمام محصولات (نسخۀ دیجیتال EPUB+NFT، اشتراک‌ها، اعتبار) با USDT روی شبکۀ پالیگان – مطابق [فصل دهم و یازدهم دامنه](./product-domain-spec.md#chapter-10).
- **تصمیم نسخۀ دیجیتال:**  
  - داخل ایران: فقط EPUB واترمارک‌شده (بدون NFT)؛  
  - خارج ایران: EPUB واترمارک‌شده + NFT روی پالیگان؛  
  - پس از تحویل دیجیتال (دانلود/ایجاد NFT)، بازپرداخت وجود ندارد – مطابق [فصل دوازدهم و سیزدهم دامنه](./product-domain-spec.md#chapter-12).
- **تصمیم لایک و Analytics:**  
  - دادهٔ عملیاتی لازم برای کارکرد لایک همیشه ذخیره می‌شود؛  
  - ثبت رویداد تحلیلی لایک در `events_behavior` فقط با Opt-in کاربر فعال می‌شود – مطابق [فصل چهارم و هفدهم دامنه](./product-domain-spec.md#chapter-4).
- **تصمیم URL کدهای پاسخ‌گو:**  
  - ساختار `/[lang]/book/v{n}/chapter/{id}/` (مثلاً `/fa/book/v1/chapter/3/`) به‌عنوان مبنای طراحی – مطابق [فصل سوم دامنه](./product-domain-spec.md#chapter-3).
- **تصمیم فرم ثبت‌نام:**  
  - فیلدهای ضروری: `email`, `display_name`, `date_of_birth`, `country`, `preferred_language`, `gender`؛  
  - فیلدهای اختیاری: `first_name`, `last_name`, `phone`, `city`;  
  - نام مستعار (display_name) ۳–۱۵ کاراکتر با قوانین ممنوعیت محتوای توهین‌آمیز/جعل هویت.
- **تصمیم بازی خلاقیت:** Idea Playground با مود Creative Clash به‌عنوان فیچر خلاقیت MVP پیاده‌سازی می‌شود؛ مودهای دیگر (مینی‌داستان، اتاق فرار کلمات) برای فاز بعد رزرو هستند – مطابق [فصل دوم و پنجم دامنه](./product-domain-spec.md#chapter-2), `technical-brief.md`.

### ۰.۲. اهداف مرحله ۵

- آماده‌سازی کامل Monorepo و محیط توسعه.
- تعریف و اعمال اسکیما در Postgres، شامل:
  - فیلدهای هویتی کاربران (نام مستعار، تاریخ تولد، کشور، زبان، جنسیت، شهر، تلفن)؛
  - `language` روی جداول محتوایی/آموزشی؛
  - مدل Critique Circles با `path_id`, `exercise_id`, `language`, `age_band` و رفتار ON DELETE مطابق Spec؛
  - ساختار `ai_feedback.payload` به صورت JSONB و score map.
- پیاده‌سازی Core Backend (ماژول‌های دامنه‌ای).
- پیاده‌سازی Worker AI با Prompt سقراطی و خروجی JSON.
- پیاده‌سازی کامل Critique Circles در سطح MVP، مطابق `feature-spec-critique-circles.md`.
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
   - مطابق سند معماری، `strict: true`، `noImplicitAny: true`، `strictNullChecks: true`.
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
     - `CRITIQUE_REQUIRED_REVIEWS=2` (N برای Participation Lock)
     - سایر متغیرها (Supabase, Sentry, …).

### ۰.۴. Observability و CI/CD

9. **راه‌اندازی استک مانیتورینگ:**
   - Prometheus:
     - Scrape کردن `/metrics` از `apps/api` و `apps/worker-*`.
   - Grafana:
     - داشبوردهای پایه:
       - Latency و Error Rate APIها (TTFB، LCP).
       - Backlog صف BullMQ (AI Worker، Publishing Worker، Crypto Worker).
   - Loki:
     - تجمیع لاگ‌ها از سرویس‌ها.
   - Sentry:
     - برای Frontend و Backend.

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
    - `capacity` (default: 3),
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

12. **افزودن ستون `language TEXT NOT NULL`** به تمام جداول محتوایی/آموزشی لازم (LearningPath, ChapterVersion, ContentItem, Exercise, Submission).

13. **تعریف ستون‌های سنی برای Critique Circles:**
    - `age_band TEXT NOT NULL CHECK (age_band IN ('teen','adult'))` در `critique_circles`.

14. **تولید و اعمال Migrationها:**
    - اجرای migration روی DB توسعه.
    - بررسی صحت FKها، خصوصاً:
      - `ON DELETE RESTRICT` برای FKهای User در Critique و Community،
      - `ON DELETE SET NULL` برای `submissions.circle_id`.

14.1. **تعریف ایندکس‌های کلیدی:**
   - برای کارایی Critique و Analytics:
     - `submissions(user_id, exercise_id)`
     - `submissions(circle_id)`
     - `circle_members(circle_id, user_id)`
     - `circle_submissions(circle_id)`
     - `peer_feedback(circle_id, reviewer_id)`
     - `peer_feedback(submission_id)`
     - `critique_circles(path_id, exercise_id, language, age_band)`
     - ایندکس روی `events_behavior(user_id, event_type, created_at)`.

14.2. **Seed داده‌های تست Critique Circles:**
   - ایجاد:
     - یک حلقۀ `open` (teen) با ۲ عضو؛
     - یک حلقۀ `active` (adult) با ۳ عضو و چند نقد؛
     - Submissionهایی با `peer_feedback_unlocked=true/false`.
   - این Seed برای تست‌های Integration در فاز ۵ استفاده می‌شود.

---

## فاز ۲: Core Backend (Monolith API)

### ۲.۱. Identity & Access Module (`apps/api/src/modules/identity`)

15. **ادغام با Supabase Auth:**
   - استفاده از `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`.
   - Middleware `auth` برای استخراج User از Session/JWT و sync با `users`.

16. **ایجاد/به‌روزرسانی پروفایل کاربر:**
   - در اولین ثبت‌نام/ورود:
     - دریافت فیلدهای ثبت‌نام (ایمیل، نام مستعار، تاریخ تولد، کشور، زبان، جنسیت) از Frontend؛
     - Validation نام مستعار (طول، کاراکترها، کلمات ممنوع)؛
     - محاسبۀ سن از `date_of_birth`:
       - اگر `age < 13` → خطا (عدم اجازهٔ ثبت‌نام)؛
       - تنظیم فیلدهای User با گروه سنی مناسب (Teen/Adult در منطق اپ، اگر لازم باشد).

17. **RBAC:**
   - Middleware `rbac`:
     - تعریف نقش‌ها و جلوگیری از دسترسی غیرمجاز.
   - نقش‌ها مطابق [فصل شانزدهم دامنه](./product-domain-spec.md#chapter-16).

---

### ۲.۲. Learning & Practice Module (`modules/learning`)

18. **Endpoints مسیرها و تمرین‌ها:**
   - `GET /paths`, `GET /paths/{id}`, `GET /exercises/{id}`.

19. **Submission Flow:**
   - `POST /paths/{pathId}/exercises/{exerciseId}/start` → ایجاد Submission (`draft`).
   - `PATCH /submissions/{id}` → Autosave.
   - `POST /submissions/{id}/submit`:
     - Validate:
       - مالکیت Submission،
       - حداقل طول متن،
       - محدودیت تعداد ارسال در روز:
         - مقدار پیش‌فرض: **۱۰ تمرین در روز**؛
         - به‌صورت Config (مثلاً `MAX_SUBMISSIONS_PER_DAY`) پیاده‌سازی شده و قابل تغییر از طریق ADR است.
     - `status='submitted'`, `submitted_at=now()`,
     - enqueue به `feedback:ai` (در صورت فعال بودن AI برای آن تمرین).

---

### ۲.۳. Feedback Module (`modules/feedback`)

20. **AI Feedback:**
   - Service `enqueueAiFeedback(submissionId)` → گذاشتن Job در `feedback:ai`.
   - `GET /submissions/{id}/feedback` → ترکیب AI + Human Feedback.

21. **Human Feedback برای منتورها:**
   - `GET /mentor/submissions` → صف مشترک Submissionهای نیازمند نقد انسانی.
   - `POST /mentor/submissions/{id}/feedback` → ثبت نقد.

---

### ۲.۴. Content Module (`modules/content`)

22. **ContentItem CRUD و Links:**
   - `GET /content-items`, `GET /content-items/{id}`،
   - Admin: `POST/PATCH/DELETE /content-items`, `POST /content-links`.

---

### ۲.۵. Community Module (`modules/community`)

این ماژول، پیاده‌سازی Critique Circles و Featured Samples را طبق `feature-spec-critique-circles.md` و [فصل هشتم/نهم دامنه](./product-domain-spec.md#chapter-8) انجام می‌دهد.

23. **Critique Circles – APIها:**
   - `POST /api/circles/join`
   - `GET /api/circles/my`
   - `POST /api/peer-feedback`
   - `POST /api/circles/report`

   منطق دقیق مطابق بخش ۴.۴ همین سند و `feature-spec-critique-circles.md` است، شامل:
   - استفاده از `(exercise_id, path_id, language, age_band)` برای یافتن/ایجاد حلقهٔ `open`,
   - محدودیت حلقه برای کاربران عادی/دارای اشتراک فعال بر اساس تعریف «حلقۀ فعال برای کاربر»،
   - جداسازی حلقه‌های Teen و Adult.

24. **Featured Samples:**
   - `GET /featured-samples` (Public).
   - Admin routes برای انتخاب/ویرایش/منتشر کردن نمونه‌ها.

---

### ۲.۶. Commerce Module (`modules/commerce`)

25. **پرداخت زرین‌پال (داخل ایران):**
   - `POST /payments/zarinpal/intents` → Intent + لینک درگاه.
   - `GET /payments/zarinpal/callback` → Verify + ثبت Transaction +:
     - فعال‌سازی `subscriptions` (در صورت خرید اشتراک)،
     - افزایش `user_credits` (در صورت خرید اعتبار نقد انسانی)،
     - ثبت خرید EPUB و راه‌اندازی Job واترمارک/DownloadToken (در صورت خرید کتاب دیجیتال).

26. **پرداخت کریپتو (خارج از ایران):**
   - `POST /payments/crypto/intents` → ثبت Intent برای USDT/Polygon (مبلغ، محصول، userId).
   - Worker Crypto:
     - Poll/Listen شبکه برای txHash؛
     - Verify مبلغ و مقصد؛
     - ثبت در `CryptoPayment`؛
     - فعال‌سازی محصول:
       - برای کتاب دیجیتال: راه‌اندازی Publishing Worker + Mint NFT؛
       - برای اشتراک/اعتبار: فعال‌سازی رکوردهای `Subscription`/`UserCredit`.

---

### ۲.۷. Compliance & Analytics Modules

27. **حذف حساب (`modules/compliance`):**
   - `POST /me/delete-account`:
     - Frontend دو گزینه را نمایش می‌دهد (ناشناس‌سازی/حذف کامل)،
     - Backend Request را در `deletion_requests` ثبت کرده و `AccountDeletionService` را اجرا می‌کند.

28. **گزارش تخلف (`modules/compliance`):**
   - `POST /reports` – فرم عمومی گزارش.
   - `GET /admin/reports` – برای ناظر حقوقی، با فیلتر بر حسب context (از جمله `critique_circle_peer_feedback`).

29. **Analytics (`modules/analytics`):**
   - Endpoint برای جمع‌آوری `events_behavior`, `events_technical` (Opt-in).
   - Jobهای Retention (حذف داده‌های قدیمی طبق سیاست دامنه).

---

## فاز ۳: Workers (AI, Publishing, Crypto)

### ۳.۱. AI Worker (`apps/worker-ai`)

30. **اتصال به Queue و DB:**
   - راه‌اندازی BullMQ روی Valkey.
   - استفاده از `packages/database/client`.

31. **Job `feedback:ai`:**
   - ورودی: `submissionId`.
   - مراحل:
     1. دریافت Submission و زبان؛
     2. ساخت Prompt سقراطی؛
     3. فراخوانی LLM؛
     4. Validate ساختار JSON مطابق Schema؛
     5. ذخیره در `AiFeedback` و update `submissions.status`.

32. **مدیریت خطا و Retry:**
   - Retry ۳ بار با Backoff؛
   - ثبت خطا در Sentry.

### ۳.۲. Publishing Worker (`apps/worker-publisher`)

33. **Watermark EPUB + DownloadToken:**
   - ورودی Job: `ebookId`, `userId`/`userEmail`.
   - گرفتن EPUB خام، واترمارک با هش ایمیل، ذخیرهٔ فایل واترمارک‌شده، ایجاد DownloadToken.

34. **اسکلت Anthology Job:**
   - تعریف Queue `publisher:anthology`، فعلاً فقط logging (پیاده‌سازی کامل در فاز بعد).

### ۳.۳. Crypto Worker (`apps/worker-crypto`)

35. **Worker Crypto/NFT:**
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

36. **صفحهٔ اصلی (`/[lang]`):**
   - معرفی پروژه،
   - نمایش بخشی از Skill Tree (آمار جمعی: تعداد تمرین‌ها، مسیرها، کاربران فعال).

37. **Learning Hub (`/[lang]/resources`):**
   - فهرست ContentItemها با فیلتر/جست‌وجو.

38. **Idea Playground (`/[lang]/playground`):**
   - مود Creative Clash:
     - سه گردونۀ تصادفی برای شخصیت/مکان/شیء،
     - تایمر ۶۰ ثانیه‌ای،
     - فیلد متنی برای Logline،
     - دکمهٔ «ثبت» و پیام تشویقی (بدون نمره).
   - اگر کاربر لاگین باشد، در فاز MVP می‌توان فقط شمارش تعداد بازی‌ها را در سمت کلاینت نگه داشت یا (در صورت نیاز) Endpoint اختیاری ساده برای ذخیرۀ جرقه‌ها اضافه کرد.

39. **صفحات کتاب (`/[lang]/book/v1/chapter/[id]`):**
   - UI فصل‌ها؛
   - فقط وقتی `NEXT_PUBLIC_ENABLE_BOOK_MODE=true` است فعال؛ در غیر این‌صورت پیام انتشار کتاب.

### ۴.۲. App کاربری

40. **Auth (`/[lang]/login`, `/[lang]/register`):**
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

41. **داشبورد (`/[lang]/dashboard`):**
   - مسیرها، لیست تمرین‌های در حال انجام و تکمیل‌شده، خلاصه رشد فردی.
   - لینک به Idea Playground برای warm-up.

42. **صفحهٔ تمرین (`/[lang]/exercise/[id]`):**
   - توضیح تمرین، ContentLinks، ادیتور متن، Autosave، Submit.
   - در صورت فعال بودن Critique، پس از Submit پیشنهاد پیوستن به حلقه.

43. **صفحهٔ بازخورد تمرین:**
   - نمایش AI Feedback + Human Feedback.

44. **صفحهٔ حلقه‌های نقد (`/[lang]/circles`):**
   - نمایش حلقۀ مرتبط با کاربر (بر اساس منطق `GET /api/circles/my`):
     - اعضا، تمرین‌های دیگران، وضعیت نقدهای user،
     - تمرین خود کاربر و وضعیت `peer_feedback_unlocked`,
   - نمایش هشدار بنری برای:
     - کاربران عادی که در یک حلقۀ فعال هستند،
     - کاربران دارای اشتراک فعال که در ≥۳ حلقۀ فعال هستند.

45. **صفحات تنظیمات و شفافیت:**
   - `/[lang]/me/settings`: مدیریت نام نمایشی، زبان، Opt-in Analytics، حذف حساب (با دو گزینه) و اطلاع صریح دربارۀ قطع دسترسی آنلاین مجدد به کتاب دیجیتال پس از حذف حساب.
   - `/[lang]/me/transparency`: خلاصهٔ داده‌ها، روند scoreهای AI، وضعیت درخواست‌های حذف/داده.

### ۴.۳. Admin UI

46. **پنل مدیر محتوا (`/[lang]/admin/content`):**
   - مدیریت فصل‌ها، ContentItemها، Links، فعال/غیرفعال کردن Critique برای تمرین‌ها.

47. **پنل منتور (`/[lang]/admin/mentors`):**
   - صف Submissionهای نیازمند نقد انسانی، Claim/Release.

48. **پنل حقوقی (`/[lang]/admin/reports`):**
   - مشاهده و فیلتر گزارش‌های تخلف، از جمله گزارش‌های Critique Circles با context خاص.

---

## فاز ۵: Edge، Discord، تست و استقرار

### ۵.۱. Cloudflare

49. **DNS و CDN:**
   - تنظیم DNS دامنهٔ اصلی،
   - فعال‌سازی CDN برای HTML/استاتیک‌ها.

50. **IP Anonymizer Worker:**
   - پیاده‌سازی Worker مطابق سند معماری (خواندن `CF-Connecting-IP`، ماسک و ارسال `X-Anonymized-IP`).

51. **WAF و Rate Limiting:**
   - پیکربندی WAF برای محافظت در برابر XSS, SQLi, …،
   - Rate Limits برای Auth, Submit, Feedback, Reports و Critique APIs و واکنش‌ها.

### ۵.۲. Discord Bot

52. **Bot Setup:**
   - ساخت Bot، دریافت Token، اتصال به سرور.

53. **ویژگی‌های MVP:**
   - Sync نقش‌های `user`/`mentor`/`premium`،
   - اعلان تمرین جدید، آماده شدن بازخورد مهم،
   - رعایت محدودیت سنی در دیسکورد:
     - کاربران ۱۳–۱۵: نقش read-only روی کانال‌های مشخص،
     - کاربران ۱۶–۱۷ و ۱۸+: امکان ارسال پیام طبق قوانین عمومی.

### ۵.۳. تست و استقرار

54. **تست‌های واحد و Integration (مسیر اصلی کاربر):**
   - ثبت‌نام (با فیلدهای هویتی) → ورود → انتخاب مسیر → start/submit تمرین → دریافت AI Feedback → پیوستن به حلقۀ نقد → نوشتن دو نقد همتا → unlock شدن نقدهای دریافتی.

55. **سناریوهای تست محدودیت حلقه‌ها (طبق توافق):**

| سناریو | کاربر | گروه سنی | عمل | نتیجهٔ مورد انتظار |
|:-------|:------|:--------|:----|:-------------------|
| ۱ | عادی | Adult | join به حلقهٔ اول (exercise X, path A) | ✅ موفق |
| ۲ | عادی | Adult | join به حلقهٔ دوم در حالی که حلقۀ اول هنوز برای او فعال است | ❌ خطای `CIRCLE_LIMIT_REACHED` |
| ۳ | دارای اشتراک فعال | Adult | join در Path A (exercise X) | ✅ موفق |
| ۴ | دارای اشتراک فعال | Adult | join در Path B (exercise Y) | ✅ موفق |
| ۵ | دارای اشتراک فعال | Adult | join دوم در همان Path A برای تمرین دیگری که Critique فعال دارد | ❌ خطا (`CIRCLE_LIMIT_REACHED_FOR_PATH`) |
| ۶ | دارای اشتراک فعال | Adult | join در Path C (سومین حلقه، در حالی که دو حلقۀ قبلی برای او فعال‌اند) | ✅ موفق + `warning=true` |
| ۷ | دارای اشتراک فعال با ۳ حلقه فعال | Adult | مشاهدهٔ `/circles` | نمایش هشدار بنری |
| ۸ | عادی | Teen | join به حلقهٔ Teen | ✅ موفق، حلقه با `age_band='teen'` ساخته می‌شود |
| ۹ | عادی | Teen | تلاش برای join به حلقهٔ Adult | ❌ جلوگیری (حلقهٔ Adult برای Teen ایجاد/استفاده نمی‌شود) |

56. **سایر تست‌های Integration:**
   - BullMQ (Retry, Failure Handling),
   - Meilisearch (جست‌وجوی FA/EN),
   - Worker Publishing (صدور Watemark EPUB و DownloadToken),
   - Crypto Worker (پایش پرداخت USDT، Mint NFT),
   - AccountDeletionService (دو حالت حذف حساب و تأثیر روی Critique/Digital Access),
   - Idea Playground – Creative Clash (تایمر، ذخیرۀ صحیح حالت، کارکرد بدون Auth و با Auth).

57. **استقرار (Deployment):**
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

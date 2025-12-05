<!-- File: architecture-fortified-monolith.md -->

# سند معماری فنی
## معماری Fortified Monolith برای پلتفرم آموزش داستان‌نویسی

**نسخه:** ۱.۱.۰  
**تاریخ مرجع:** جمعه، ۲۸ نوامبر ۲۰۲۵  

این سند، معماری فنی نهایی پلتفرم را توصیف می‌کند؛  
با در نظر گرفتن تمام تصمیم‌ها و R&D انجام‌شده روی معماری، محتوا، AI، دوزبانگی، Book Mode، حلقه‌های نقد (Critique Circles)، سیاست‌های حریم خصوصی/حذف داده و فیچرهای خلاقیت (Idea Playground / Creative Clash).  

این سند در هماهنگی با:

- **سند دامنهٔ ۳۳ فصلی** (`product-domain-spec.md`)  
- بریف فنی (`technical-brief.md`)  
- و Spec فیچر Critique Circles (`feature-spec-critique-circles.md`)

نوشته شده است.  
هر طراحی، پیاده‌سازی و تصمیم فنی باید با این سند منطبق باشد و هر تغییر اساسی نیازمند ADR مستقل است.

---

## ۱. اصول و فلسفهٔ معماری

۱. **Monolith ماژولار (Fortified Monolith):**
   - یک سرویس Backend واحد (یک دیپلوی) با ماژول‌های دامنه‌ای واضح.
   - هیچ Microservice جدا با دیتابیس مستقل برای Auth/LMS/Payment… وجود ندارد.
   - جداشدگی فقط برای Workerهای ناهمگام (AI Feedback, Publishing, Crypto/NFT) است.
   - این رویکرد:
     - حذف حساب و اعمال سیاست‌های حریم خصوصی تعریف‌شده در [فصل هفدهم سند دامنه](./product-domain-spec.md#chapter-17) را ساده‌تر می‌کند،
     - ریسک پیچیدگی زودهنگام و هزینهٔ عملیاتی را در مقیاس فعلی کاهش می‌دهد.

۲. **طراحی حول دامنه (Domain-Centric Design):**
   - Contextهای دامنه‌ای اصلی:
     - Identity & Access
     - Learning & Practice
     - Feedback (AI & Human)
     - Content & Chapters & ContentItems
     - Community & Featured (Critique Circles, Featured Samples)
     - Commerce & Billing
     - Compliance & Data Governance
     - Analytics & Events
   - این Contextها در یک کدبیس و یک پایگاه دادهٔ مشترک پیاده می‌شوند (Monolith)، نه Microserviceهای جداگانه.
   - مدل مفهومی داده با [فصل بیست‌وششم سند دامنه](./product-domain-spec.md#chapter-26) منطبق است؛ از جمله:
     - جداسازی داده‌های هویتی (ایمیل، نام مستعار، تاریخ تولد، کشور، زبان، جنسیت)،
     - مدل Critique Circles بر اساس مسیر، تمرین، زبان و بازۀ سنی (teen/adult).

۳. **Fortification (ایزولاسیون پردازش‌های سنگین):**
   - پردازش‌های سنگین/طولانی (AI، واترمارک EPUB، Crypto/NFT، تولید Anthology) در Workerهای ایزوله اجرا می‌شوند.
   - این Workerها پروسه/کانتینر مستقل از وب‌سرور دارند و از طریق Queue روی Valkey تغذیه می‌شوند.
   - وب‌سرور زیر بار سنگین AI یا Crypto کند نمی‌شود و قابلیت Retry و مانیتورینگ جداگانه وجود دارد.

۴. **Edge-First برای عملکرد و حریم خصوصی:**
   - Cloudflare در جلو:
     - CDN برای HTML و استاتیک‌ها،
     - WAF برای امنیت،
     - Workers برای ناشناس‌سازی IP و منطق سبک لبه‌ای (به‌ویژه فصل شانزدهم کتاب و برخی تعاملات عمومی مانند لایک بدون حساب).
   - Edge:
     - بخشی از بار را از Origin می‌گیرد،
     - اجازه می‌دهد سیاست‌های حریم خصوصی (مثل IP Anonymization) قبل از رسیدن درخواست به Backend اعمال شوند (مطابق [فصل هفدهم](./product-domain-spec.md#chapter-17)).

۵. **Single Source of Truth برای داده:**
   - PostgreSQL تنها منبع حقیقت است.
   - Meilisearch، Valkey و S3 نقش مکمل دارند (Search/Cache/Storage)، نه منبع دادهٔ اصلی.
   - تمام تصمیم‌ها در مورد حریم خصوصی، حذف حساب و ناشناس‌سازی، حول داده‌های Postgres طراحی می‌شوند.

۶. **دو‌زبانگی ذاتی (Native i18n):**
   - معماری Front و DB از روز اول برای `language = 'fa' | 'en'` طراحی شده است.
   - Routing (`/fa/...` و `/en/...`) و ستون `language` در جداول محتوایی، از ابتدا وجود دارند؛
   - rollout محتوای انگلیسی به‌صورت تدریجی انجام می‌شود، بدون نیاز به تغییر در معماری (مطابق [فصل بیست‌ودوم سند دامنه](./product-domain-spec.md#chapter-22)).

۷. **AI سقراطی (Socratic AI):**
   - AI نقش «مربی پرسش‌گر» دارد، نه مصحح متن یا معلم دبستان.
   - خروجی AI به‌صورت JSON ساختاریافته با تأکید بر مشاهدات توصیفی، سؤال‌ها و پیشنهادهای تجربی است، نه تصحیح خط‌به‌خط.
   - ساختار امتیازدهی (score) به صورت نگاشت `metric_name → value` است تا قابلیت توسعه در آینده حفظ شود؛ در MVP سه معیار اصلی استفاده می‌شود (مطابق [فصل ششم سند دامنه](./product-domain-spec.md#chapter-6)).

۸. **Book Mode با Feature Flag:**
   - ساختار فصل‌ها و مسیر `/[lang]/book/v{n}/chapter/[id]` در دامنه از آغاز موجود است،
   - اما تا زمان چاپ، در UI پشت Feature Flag خاموش است و تجربهٔ کاربر روی «مسیرهای آموزشی آنلاین + مرکز یادگیری + Idea Playground» متمرکز می‌ماند ([فصل سوم](./product-domain-spec.md#chapter-3)).

۹. **Compliance-Driven Design (طراحی مبتنی بر حاکمیت داده):**
   - معماری از ابتدا براساس اصول حریم خصوصی، حذف حساب دوحالته (ناشناس‌سازی/حذف کامل)، ناشناس‌سازی داده‌های جامعه‌محور، **حذف پیوندهای هویتی (با SET NULL روی FKهای کاربر در Critique/Community)** و شفافیت داده‌ای طراحی می‌شود (هماهنگ با [فصل هفدهم و هجدهم](./product-domain-spec.md#chapter-17)).
   - رفتارهای حساس (مانند حذف/ناشناس‌سازی مشارکت‌ها در Critique Circles) توسط سرویس‌های دامنه‌ای (AccountDeletionService و …) کنترل می‌شوند، نه توسط `ON DELETE CASCADE`های خام در دیتابیس.

۱۰. **Observability-First:**
    - نظارت بر سیستم (خطاها، متریک‌ها، لاگ‌ها) از ابتدا جزئی از معماری است، نه افزودنی بعدی.
    - استک محوری: Prometheus + Grafana + Loki + Sentry.

---

## ۲. پشتهٔ تکنولوژیک (Tech Stack)

### ۲.۱. Frontend

- **Next.js** 16.0.4 (App Router)
- **React** 19.2
- **TypeScript** (Strict Mode)
- CSS/Design System:
  - Tailwind یا معادل، RTL-ready، با پشتیبانی از حالت تیره/روشن.
  - طراحی مبتنی بر Design System داخلی، با در نظر گرفتن A11y و RTL/LTR.

- کتاب‌خوان EPUB:
  - epub.js / Readium Web با پشتیبانی از راست‌به‌چپ.

### ۲.۲. Backend Core

- **Node.js** 24.11.1 (LTS)
- فریم‌ورک HTTP:
  - Fastify یا Express (Monolith واحد)
- API:
  - REST/JSON با مستندسازی OpenAPI/Swagger
- زبان:
  - TypeScript سخت‌گیرانه (`strict: true`)
- ساختار:
  - معماری لایه‌ای (Routing → Controllers → Services → Repos → DB),
  - ماژول‌بندی حول Contextهای دامنه‌ای ذکرشده در بخش ۱.

### ۲.۳. Data & Infra

- **Database:** PostgreSQL 18.1
- **Cache & Queue & Rate Limit:** Valkey 9.0.0 (Redis-compatible)
- **Search:** Meilisearch 1.27.0
- **Object Storage:** سازگار با S3 (Cloudflare R2 / MinIO)

### ۲.۴. Edge & Security

- **Cloudflare:**
  - DNS
  - CDN
  - WAF
  - Workers (برای IP Anonymizer و منطق سبک Edge)

### ۲.۵. Auth & Community

- **Supabase Auth:**
  - Email+Password, Discord OAuth
  - امکان افزودن MFA برای نقش‌های حساس در آینده.
- **Discord Bot:** Node.js، برای:
  - نقش‌دهی،
  - اعلان‌ها،
  - قلاب برای آرشیو گفتگوها و همگام‌سازی سطحی نقش‌ها (مطابق [فصل هشتم سند دامنه](./product-domain-spec.md#chapter-8)).

### ۲.۶. AI & Side Services

- **LLM**:
  - Claude 3.5 Sonnet / GPT‑4o (انتخاب نهایی بسته به کیفیت، هزینه و سیاست داده).
- **AI Worker**:
  - Node.js + TS، برای:
    - AI Feedback،
    - (در آینده) Muse Agent.
- **Crypto Worker**:
  - Node.js (یا Go در صورت نیاز)، برای:
    - پایش پرداخت‌های USDT روی Polygon،
    - Mint NFT برای نسخۀ دیجیتال کتاب کاربران خارج از ایران،
    - سایر عملیات Crypto در فازهای بعدی.
- **Publishing Worker**:
  - Node.js برای واترمارک EPUB، صدور DownloadToken و (در آینده) ساخت Anthology.

### ۲.۷. Observability & Monitoring

- **Error Tracking:** Sentry برای Frontend و Backend
- **Metrics:** Prometheus
- **Dashboards:** Grafana
- **Logs:** Loki

---

## ۳. لایه‌ها و مرزهای سیستم

### ۳.۱. User & Edge

```text
Browser → Cloudflare (CDN/WAF/Workers) → Origin (Next.js + API)
```

- Cloudflare:
  - تحویل HTML و استاتیک‌ها از نزدیک‌ترین Edge،
  - Cache صفحات SSG/ISR،
  - Rate Limiting در سطح لبه برای مسیرهای حساس (Auth, Submit, Feedback, Reports, **Critique APIs**, واکنش‌های عمومی).

- Cloudflare Workers:
  - **IP Anonymizer:**
    - خواندن IP از `CF-Connecting-IP`,
    - صفر کردن octet آخر (`x.y.z.0`),
    - اضافه کردن `X-Anonymized-IP` به درخواست ارسالی به Backend،
    - Backend فقط این IP ناشناس‌شده را در لاگ‌ها/آنالیتیکس ذخیره می‌کند.
  - (در آینده) منطق سبک فصل ۱۶:
    - مثلاً مدیریت Fingerprint و Rate Limit واکنش‌های بدون حساب از لبه.

### ۳.۲. Web App Layer (Next.js 16)

- ارائهٔ تمام UI:
  - **Public:**  
    - `/[lang]/` – صفحهٔ اصلی
    - `/[lang]/resources` – مرکز یادگیری (ContentItems)
    - `/[lang]/playground` – Idea Playground (Creative Clash و مودهای بعدی)
    - `/[lang]/samples` – نمونه‌های برگزیده
    - `/[lang]/book/v1/chapter/[id]` – فصل‌ها (پشت Book Mode Flag)
    - `/[lang]/legal/*` – سیاست‌ها و اسناد حقوقی (از جمله نسخهٔ ساده/کامل سیاست داده مطابق [فصل هجدهم](./product-domain-spec.md#chapter-18))
  - **App (کاربری):**
    - `/[lang]/dashboard` – داشبورد مسیرها و تمرین‌ها
    - `/[lang]/exercise/[id]` – صفحهٔ تمرین
    - `/[lang]/circles` – حلقه‌های نقد (Critique Circles)
    - `/[lang]/me/settings` – تنظیمات حساب، حریم خصوصی و سناریوی حذف حساب
    - `/[lang]/me/transparency` – داشبورد شفافیت داده و رشد فردی
  - **Admin:**
    - `/[lang]/admin/content` – مدیریت فصل‌ها و ContentItemها
    - `/[lang]/admin/mentors` – صف تمرین‌های منتورها
    - `/[lang]/admin/reports` – گزارش‌های تخلف

- i18n:
  - `locales: ['fa', 'en']`, `defaultLocale: 'fa'`.
  - فایل‌های ترجمه در `apps/web/locales/fa/*.json` و `en/*.json`.
  - Root Layout:
    - `dir="rtl"` و `lang="fa"` برای fa،
    - `dir="ltr"` و `lang="en"` برای en.

- Book Mode:
  - Flag: `NEXT_PUBLIC_ENABLE_BOOK_MODE`.
  - اگر false:
    - لینک‌ها و منوی کتاب/فصل‌ها مخفی‌اند،
    - تلاش برای بازکردن `/book/...` پیامی محترمانه نشان می‌دهد («کتاب هنوز منتشر نشده است»).

---

## ۴. Core Application (Monolith Node.js)

ساختار کد:

```text
apps/api/
└─ src/
   ├─ main.ts
   ├─ server.ts
   ├─ config/
   ├─ infra/
   ├─ middleware/
   └─ modules/
      ├─ identity/
      ├─ learning/
      ├─ feedback/
      ├─ content/
      ├─ community/
      ├─ commerce/
      ├─ compliance/
      └─ analytics/
```

### ۴.۱. Identity & Access

- مسئول:
  - ادغام Supabase Auth با مدل کاربری داخلی،
  - نگهداری پروفایل‌ها و نقش‌ها،
  - اعمال RBAC و اصل کمترین امتیاز (مطابق [فصل شانزدهم](./product-domain-spec.md#chapter-16))،
  - مدیریت فیلدهای هویتی لازم برای سیاست سن و کشور و زبان.

- جداول:
  - `users` (خلاصهٔ فیلدهای کلیدی):
    - `id`: UUIDv7
    - `email`: TEXT, UNIQUE, NOT NULL
    - `display_name`: TEXT, NOT NULL (۳ تا ۱۵ کاراکتر، Validation در سطح اپ)
    - `date_of_birth`: DATE, NOT NULL (برای محاسبۀ سن و بازۀ سنی teen/adult)
    - `country`: TEXT, NOT NULL (کد کشور، مثل `IR`)
    - `preferred_language`: TEXT NOT NULL (`'fa' | 'en'`)
    - `gender`: TEXT NOT NULL (`'female' | 'male' | 'non_binary' | 'prefer_not_to_say'`)
    - `city`: TEXT NULL
    - `phone`: TEXT NULL
    - `created_at`, `updated_at`
  - `roles`, `user_roles`:
    - نقش‌ها: `user`, `mentor`, `content_manager`, `ops_manager`, `finance`, `legal`, `admin`.

- Middleware:
  - `auth`: اعتبارسنجی Session/JWT Supabase و استخراج User Context.
  - `rbac`: بررسی نقش‌ها و اعمال اصل حداقل دسترسی روی Endpointها.

- محاسبۀ بازۀ سنی:
  - در لایهٔ سرویس، بر اساس `date_of_birth`، دو گروه سنی استخراج می‌شوند:
    - `teen`: سن ۱۳ تا ۱۷ سال،
    - `adult`: ۱۸ سال و بالاتر.
  - اگر سن کاربر < ۱۳ سال محاسبه شود، اجازهٔ ایجاد حساب داده نمی‌شود.

---

### ۴.۲. Learning & Practice

- جداول:
  - `learning_paths`:
    - نمایندۀ مسیرها (مثلاً «از ایده تا طرح کوتاه»), با `language`.
  - `sessions`:
    - جلسات هر مسیر، با `order`.
  - `exercises`:
    - تمرین‌ها، با فیلدهای:
      - `path_id`, `session_id`, `title`, `description`, `language`, `is_critique_enabled`, …
  - `submissions`:
    - `id`, `user_id`, `exercise_id`, `language`,
    - `draft_content`, `final_content`,
    - `status` (`draft`, `submitted`, `ai_reviewed`, `human_reviewed`, …),
    - `submitted_at`,
    - `circle_id` (denormalized FK به `critique_circles.id` با `ON DELETE SET NULL`),
    - `peer_feedback_unlocked` (bool),
    - `created_at`, `updated_at`.

- APIهای اصلی:
  - `GET /paths`, `GET /paths/{id}`, `GET /exercises/{id}`.
  - `POST /paths/{pathId}/exercises/{exerciseId}/start`.
  - `PATCH /submissions/{id}` (Autosave).
  - `POST /submissions/{id}/submit`.

Idea Playground (Creative Clash و مودهای دیگر) در این Context به‌عنوان «فعالیت‌های خلاقیت جانبی» در نظر گرفته می‌شود:

- نیازی به جدول جدید برای خود بازی در MVP نیست؛
- در صورت تمایل به ذخیرۀ جرقه‌ها، می‌توان جدولی مانند `idea_sparks` با فیلدهای `user_id`, `prompt`, `logline`, `created_at` در این Context اضافه کرد.

---

### ۴.۳. Feedback (AI & Human)

#### ۴.۳.۱. AI Feedback

- Worker AI در `apps/worker-ai`‌ queue `feedback:ai` را مصرف می‌کند.
- برای هر Submission:
  - متن + `language` از DB خوانده می‌شود.
  - Prompt سقراطی ساخته می‌شود.
  - LLM صدا زده می‌شود.
  - خروجی در JSON ساختاریافته مانند زیر (نمونه) برمی‌گردد:

    ```json
    {
      "overall_impression": "...",
      "strengths": ["..."],
      "areas_for_exploration": ["..."],
      "questions_to_consider": ["..."],
      "suggested_experiments": ["..."],
      "score": {
        "narrative_clarity": 3,
        "character_depth": 4,
        "language_use": 3
      }
    }
    ```

  - در DB:
    - `ai_feedback.payload` از نوع `JSONB` است و حاوی این ساختار است.
    - فیلد `score` یک آبجکت JSON درون `payload` است که به صورت Map `metric_name → number` در نظر گرفته می‌شود؛ در MVP کلیدهای آن سه مقدار فوق است.

- Endpoint:
  - `GET /submissions/{id}/feedback`:
    - AI Feedback + Human Feedback (در صورت وجود) را برمی‌گرداند.

#### ۴.۳.۲. Human Feedback

- جداول:
  - `human_feedback`:
    - `id`, `submission_id`, `mentor_id`, `body`, `created_at`, …
- Flow:
  - منتورها از پنل خود، صف Submissionهای مورد نیاز را می‌بینند (صف مشترک، Pull-based).
  - با `POST /mentor/submissions/{id}/feedback` نقد را ثبت می‌کنند.
  - هیچ تخصیص خودکار Round-robin در MVP وجود ندارد؛ هر منتور آیتم را Claim می‌کند.

---

### ۴.۴. Content & Chapters & ContentItems

#### ۴.۴.۱. Chapters

- جداول:
  - `chapters`:
    - اطلاعات کلی فصل (شماره، slug، …).
  - `chapter_versions`:
    - نسخه‌های محتوایی فصل، شامل:
      - `chapter_id`, `language`, `book_version`, `title`, `body`, `status`, `is_current`, `created_at`, …

- رفتار:
  - در دامنه از آغاز وجود دارند.
  - در UI:
    - تا زمانی که Book Mode خاموش است، نمایش داده نمی‌شوند (یا صفحهٔ انتظار چاپ نشان می‌دهند).
  - برای هر `(chapter_id, language, book_version)`:
    - می‌تواند چند نسخه `status='published'` در تاریخچه وجود داشته باشد،
    - اما فقط یکی از آن‌ها `is_current = true` است که به `/[lang]/book/v{n}/chapter/{id}` نگاشت می‌شود (همسو با [فصل سوم سند دامنه](./product-domain-spec.md#chapter-3)).

#### ۴.۴.۲. ContentItems و ContentLinks

- جداول:

```sql
content_items (
  id          UUID PRIMARY KEY,
  title       TEXT,
  summary     TEXT NULL,
  body        TEXT NULL,
  url         TEXT NULL,
  media_type  TEXT NOT NULL,  -- article, video, audio, image, diagram, slide, epub, ...
  tags        TEXT[] NOT NULL DEFAULT '{}',
  language    TEXT NOT NULL,  -- 'fa' | 'en'
  status      TEXT NOT NULL,  -- 'draft' | 'published'
  created_at  TIMESTAMPTZ,
  updated_at  TIMESTAMPTZ
);

content_links (
  id           UUID PRIMARY KEY,
  content_id   UUID NOT NULL REFERENCES content_items(id) ON DELETE CASCADE,
  target_type  TEXT NOT NULL, -- 'path' | 'session' | 'exercise' | 'chapter'
  target_id    UUID NOT NULL,
  relationship TEXT NOT NULL, -- 'supplemental' | 'example' | 'further-reading' | ...
  created_at   TIMESTAMPTZ
);
```

- پیش از چاپ کتاب:
  - ContentItemها برای منابع عمومی (Learning Hub) و محتوای تکمیلی مسیرها/تمرین‌ها استفاده می‌شوند (`target_type='path/session/exercise'`).
- بعد از چاپ کتاب:
  - همان ContentItemها می‌توانند با `target_type='chapter'` به فصل‌ها لینک شوند و نقش محتوای تکمیلی فصل را بگیرند.

---

### ۴.۵. Community & Featured

#### ۴.۵.۱. Critique Circles (حلقه‌های نقد)

این بخش پیاده‌سازی فنی فیچر «حلقه‌های نقد (Critique Circles)» است که در [فصل هشتم سند دامنه](./product-domain-spec.md#chapter-8) تعریف دامنه‌ای آن آمده و در `feature-spec-critique-circles.md` به تفصیل فنی تشریح شده است.

- جداول (با رفتار ON DELETE به‌صراحت):

```sql
critique_circles (
  id          UUID PRIMARY KEY,
  name        TEXT NULL,
  status      TEXT NOT NULL,    -- 'open' | 'active' | 'closed'
  capacity    INTEGER NOT NULL, -- پیش‌فرض: 3
  language    TEXT NOT NULL,    -- 'fa' | 'en'
  path_id     UUID NOT NULL REFERENCES learning_paths(id),
  exercise_id UUID NOT NULL REFERENCES exercises(id),
  age_band    TEXT NOT NULL,    -- 'teen' | 'adult'
  owner_id    UUID NULL REFERENCES users(id) ON DELETE RESTRICT,
  created_at  TIMESTAMPTZ NOT NULL,
  updated_at  TIMESTAMPTZ NOT NULL
);

circle_members (
  id          UUID PRIMARY KEY,
  circle_id   UUID NOT NULL REFERENCES critique_circles(id) ON DELETE CASCADE,
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  role        TEXT NOT NULL,    -- 'member' | 'owner'
  joined_at   TIMESTAMPTZ NOT NULL,
  left_at     TIMESTAMPTZ NULL,
  UNIQUE(circle_id, user_id)
);

circle_submissions (
  id            UUID PRIMARY KEY,
  circle_id     UUID NOT NULL REFERENCES critique_circles(id) ON DELETE CASCADE,
  submission_id UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
  author_id     UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  created_at    TIMESTAMPTZ NOT NULL,
  UNIQUE(circle_id, submission_id)
);

peer_feedback (
  id            UUID PRIMARY KEY,
  circle_id     UUID NOT NULL REFERENCES critique_circles(id) ON DELETE CASCADE,
  submission_id UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
  reviewer_id   UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  body          TEXT NOT NULL,
  score         INTEGER NULL,
  created_at    TIMESTAMPTZ NOT NULL,
  UNIQUE(circle_id, submission_id, reviewer_id)
);
```

- نکات مهم:
  - هر حلقه از نظر دامنه‌ای:
    - برای یک مسیر مشخص (`path_id`)،
    - یک تمرین مشخص (`exercise_id`)،
    - یک زبان مشخص (`language`)،
    - و یک بازۀ سنی مشخص (`age_band = 'teen' | 'adult'`) تشکیل می‌شود.
  - `circle_submissions` منبع اصلی (canonical) اتصال Submission به حلقه است؛
  - فیلد `submissions.circle_id` یک shortcut denormalized است (`ON DELETE SET NULL`) که باید در سطح اپلیکیشن منطبق با `circle_submissions` نگه داشته شود.
  - ظرفیت پیش‌فرض هر حلقه: ۳ (قابل پیکربندی).
  - هر حلقه برای یک `exercise_id` و یک `path_id` و یک `language` و یک `age_band` است؛ Matching join بر همین اساس است.
  - هر کاربر در هر حلقه حداکثر یک Submission دارد.
  - owner در MVP فقط سازندهٔ حلقه است و در MVP نقش مدیریتی ویژه‌ای ندارد؛ این فیلد برای توسعه‌های آتی (بستن دستی حلقه، مدیریت اعضا، deadline) رزرو شده است.

- محدودیت‌ها و منطق تجاری (هم‌سو با Spec و سند دامنه):
  - کاربر عادی:
    - در هر لحظه حداکثر ۱ حلقۀ فعال (بر اساس تعریف «فعال برای او») در کل سیستم.
  - کاربر دارای اشتراک فعال:
    - در هر Path حداکثر ۱ حلقۀ فعال؛
    - اگر در دو حلقۀ فعال باشد و برای سومی Join کند، Join مجاز است و Backend فلگ هشدار (`warning=true`) برمی‌گرداند.
  - **تعریف «حلقۀ فعال برای کاربر» در منطق محدودیت:**
    - حلقه‌ای که:
      - کاربر عضو آن است (`circle_members.left_at IS NULL`)،
      - و برای Submission خودش در آن حلقه هنوز `peer_feedback_unlocked = false` است.
    - پس از این‌که کاربر حداقل N نقد معتبر (در MVP: N=2) روی تمرین‌های دیگران در همان حلقه نوشت و نقدهای همتا روی تمرین خودش unlock شد، این حلقه دیگر برای او در منطق محدودیت به‌عنوان «فعال» شمرده نمی‌شود؛ حتی اگر `critique_circles.status` همچنان `active` باشد.
  - `age_band`:
    - از روی `users.date_of_birth` محاسبه می‌شود:
      - ۱۳–۱۷ → `age_band='teen'`,
      - ۱۸+ → `age_band='adult'`.
    - کاربران `teen` فقط در حلقه‌های teen، و کاربران `adult` فقط در حلقه‌های adult قرار می‌گیرند.
  - N (تعداد نقد لازم برای unlock): ۲، مقدار ثابت در Config (مثلاً ENV `CRITIQUE_REQUIRED_REVIEWS=2`).
  - حداقل طول `peer_feedback.body`: ۲۰۰ کاراکتر؛ حداکثر ۵۰۰۰ کاراکتر در Validation Backend اعمال می‌شود.

- مدیریت Race Condition در Join:
  - هنگام جست‌وجو/ایجاد حلقۀ `open` برای اضافه‌کردن عضو جدید:
    - از Transaction و Row-level Lock روی رکورد حلقه استفاده می‌شود تا در سناریوی هم‌زمانی، ظرفیت بیش از حد پر نشود.
    - در صورت پر شدن ظرفیت در لحظهٔ نهایی، کاربر به حلقۀ دیگری هدایت یا حلقۀ جدیدی برای او ساخته می‌شود (طبق منطق `feature-spec-critique-circles.md`).

- APIها (به‌طور خلاصه، شرح کامل در `feature-spec-critique-circles.md`):
  - `POST /api/circles/join` – پیوستن به حلقه، با در نظر گرفتن `(exercise_id, language, path_id, age_band)` و محدودیت‌ها.
  - `GET /api/circles/my` – مشاهده حلقهٔ فعال.
  - `POST /api/peer-feedback` – ثبت نقد همتا.
  - `POST /api/circles/report` – گزارش تخلف در حلقه (نگاشت به `reports` با context مناسب).

- حریم خصوصی و حذف حساب:
  - هیچ FK از این جداول به `users(id)` دارای `ON DELETE CASCADE` نیست؛
  - در زمان حذف حساب، `AccountDeletionService` براساس انتخاب کاربر (ناشناس‌سازی/حذف کامل) این رکوردها را اصلاح یا حذف می‌کند؛
    - در حالت ناشناس‌سازی، FKهای `user_id`/`author_id`/`reviewer_id` به `NULL` ست می‌شوند و متن‌ها باقی می‌مانند؛
    - در حالت حذف کامل، Submissionها و نقدهای مرتبط (و معادل آن‌ها در Critique Circles) حذف می‌شوند، با این آگاهی که نقدهای دیگران روی تمرین‌های حذف‌شده هم ممکن است از بین برود.
  - این رفتار با [فصل هفدهم سند دامنه](./product-domain-spec.md#chapter-17) هم‌خوان است.

#### ۴.۵.۲. Featured Samples

- جداول:
  - `featured_candidates`, `featured_samples`, `consents`.

- Flow:
  - منتورها نمونه‌های خوب را علامت‌گذاری می‌کنند (Candidate).
  - الگوریتم روی before/after و امتیازهای AI پیشنهاد می‌دهد.
  - Content Manager تأیید می‌کند.
  - رضایت نویسنده ثبت می‌شود (Consent + PDF).
  - نمونه‌ها در سایت عمومی نمایش داده می‌شوند.

---

### ۴.۶. Commerce & Billing

- جداول:
  - `transactions`
  - `subscriptions`
  - `user_credits`
  - `crypto_payments`
  - `wallet_token_links`
  - `download_tokens` (برای لینک‌های دانلود EPUB واترمارک‌شده).

- تشخیص «کاربر دارای اشتراک فعال»:
  - وجود حداقل یک رکورد در `subscriptions` با:
    - `status = 'active'`
    - `expires_at > NOW()`

- پرداخت داخل ایران (زرین‌پال):
  - Flow:
    - Intent → Redirect → Callback → Verify → ثبت تراکنش → فعال‌سازی اشتراک/اعتبار/دسترسی به کتاب دیجیتال (EPUB).

- پرداخت خارجی و NFT (فعال در MVP برای کاربران خارج از ایران):
  - Crypto Worker مسئول:
    - پایش تراکنش‌های USDT/Polygon (از طریق API/Node Provider)،
    - ثبت `crypto_payments` پس از Verify مبلغ و مقصد،
    - فعال‌سازی اشتراک/اعتبار یا آغاز فرآیند صدور نسخۀ دیجیتال (EPUB+NFT).
  - برای خرید نسخۀ دیجیتال کتاب:
    - پس از تأیید پرداخت:
      - Publishing Worker EPUB را واترمارک کرده و DownloadToken ایجاد می‌کند؛
      - Crypto Worker NFT را Mint می‌کند و در `wallet_token_links` ثبت می‌کند.

- Download Tokens:
  - برای هر خرید کتاب دیجیتال، Worker Publishing:
    - EPUB خام را می‌گیرد، هش ایمیل را درج می‌کند، فایل واترمارک‌شده را در Storage ذخیره می‌کند و DownloadToken تولید می‌کند.
  - کاربر می‌تواند هر زمان از طریق حساب خود، توکن جدید بگیرد و فایل را مجدداً دانلود کند، تا زمانی که حسابش فعال است.

---

### ۴.۷. Compliance & Data Governance

- مسئولیت‌ها:
  - حذف حساب (AccountDeletionService),
  - مدیریت رضایت‌ها (Consents),
  - ثبت و نگهداری رخدادهای امنیتی،
  - اعمال Retention،
  - ناشناس‌سازی مشارکت‌های جامعه‌محور مطابق سیاست دامنه.

- جداول:
  - `deletion_requests`
  - `security_events`
  - `consents`
  - `reports` (گزارش تخلف)

- AccountDeletionService (همسو با [فصل هفدهم](./product-domain-spec.md#chapter-17)):
  - بر اساس انتخاب کاربر دو حالت را مدیریت می‌کند:
    1. ناشناس‌سازی مشارکت‌ها:
       - حذف رکورد `users`,
       - برای جداول جامعه‌محور (مانند Critique Circles، Peer Feedback و…):
         - ست‌کردن FKهای `user_id`/`author_id`/`reviewer_id` به `NULL`,
       - حفظ متن تمرین‌ها و نقدها در سیستم، بدون هیچ پیوند قابل‌بازسازی با هویت کاربر.
    2. حذف کامل مشارکت‌ها:
       - حذف Submissionها و نقدهای مرتبط؛
       - به‌دلیل `ON DELETE CASCADE` روی `submission_id` در `peer_feedback`, نقدهای دیگران روی تمرین‌های حذف‌شده نیز پاک می‌شوند.
  - در هر دو حالت، سوابق مالی، رخدادهای امنیتی و داده‌های موردنیاز برای پیشگیری از تقلب طبق الزامات قانونی و سیاست نگهداری داده حفظ می‌شوند.

---

### ۴.۸. Analytics & Events

- جداول:
  - `events_behavior` (Opt-in),
  - `events_technical`.

- داده‌های رفتاری:
  - `page_view`, `exercise_start/save/submit`, `feedback_read`, `search`, `qrcode_scan`, `web_vitals`, …
  - `reaction_like` به‌عنوان رویداد تحلیلی اختیاری (در کنار دادهٔ عملیاتی لایک که در ساختارهای دیگری برای کارکرد خود فیچر نگه‌داری می‌شود).

- Opt-in:
  - پیش‌فرض خاموش؛ فقط با رضایت کاربر فعال می‌شود.
  - تنظیمات در `/me/settings` (مطابق [فصل هفدهم](./product-domain-spec.md#chapter-17)) قابل تغییر است.

- Retention:
  - ۳۰ روز برای رویدادهای رفتاری،
  - ۹۰ روز برای رویدادهای فنی.

- **Skill Tree Aggregation:**
  - انجین Skill Tree (مطابق [فصل نوزدهم سند دامنه](./product-domain-spec.md#chapter-19) و بریف فنی) از روی داده‌های زیر آمار تجمیعی ساده تولید می‌کند:
    - `submissions` (برای شمارش تمرین‌ها و مسیرهای کامل‌شده)،
    - `ai_feedback.payload.score` (برای محاسبات aggregate غیرشخصی، در صورت نیاز فازهای بعدی)،
    - `events_behavior` (برای تعیین کاربران فعال).
  - **هیچ جدول جداگانه‌ای برای Leaderboard فردی طراحی نشده است.**  
    آمارهای فردی (trend امتیازها، نمودار پیشرفت) فقط در داشبورد کاربر نمایش داده می‌شوند و برای مقایسه با دیگران استفاده نمی‌شوند.

---

## ۵. Data Layer

### ۵.۱. PostgreSQL 18.1

- منبع حقیقت برای:
  - کاربران، نقش‌ها، محتوای کتاب و ContentItems،
  - مسیرها، تمرین‌ها، submissions،
  - بازخوردها، حلقه‌های نقد، نمونه‌های برگزیده،
  - معاملات مالی، رضایت‌ها، گزارش‌ها، رویدادها.
- استفاده توصیه‌شده:
  - `uuidv7()` به‌عنوان کلید اصلی برای جداول پرتعداد،
  - استفاده از ایندکس‌های مناسب برای queries جست‌وجو/گزارش،
  - عدم اتکا به Cascade برای سناریوهای پیچیدهٔ حریم خصوصی؛ تمام `ON DELETE`های مرتبط با `users(id)` در جداول جامعه‌محور از نوع `RESTRICT` هستند و حذف/ناشناس‌سازی از طریق Application انجام می‌شود؛
  - در مقابل، برای وابستگی‌هایی مثل `peer_feedback.submission_id` از `ON DELETE CASCADE` استفاده می‌شود تا هنگام حذف کامل مشارکت‌ها، وابستگی‌ها به‌درستی پاک شوند.

- **ساختار `ai_feedback.payload`:**
  - نوع ستون: `JSONB`.
  - ساختار داخلی: یک JSON با فیلدهایی شامل:
    - `overall_impression`, `strengths`, `areas_for_exploration`, `questions_to_consider`, `suggested_experiments`, `score`.
  - `score`: یک آبجکت JSON (`{ "metric_name": number, ... }`). در MVP سه کلید: `narrative_clarity`, `character_depth`, `language_use`.

### ۵.۲. Meilisearch 1.27.0

- ایندکس‌ها:
  - `chapters_index`,
  - `content_items_index`,
  - `featured_samples_index`,
  - (در صورت نیاز) `glossary_index`.

- امکانات:
  - جست‌وجوی full-text با typo tolerance،
  - faceting/filtering (مثلاً بر اساس زبان، برچسب‌ها).
- برای فارسی:
  - نیاز به normalize متن (حذف نیم‌فاصلهٔ بد، یکسان‌سازی حروف، …) در مرحلهٔ index.

### ۵.۳. Valkey 9.0.0

- استفاده‌ها:
  - Cache:
    - نتایج queries پرتکرار،
    - آمار Skill Tree و داشبوردهای کلی.
  - Queue:
    - `feedback:ai`,
    - `publisher:epub`,
    - `crypto:verify`,
    - queues دیگر در آینده.
  - Rate Limiting:
    - شمارنده برای هر IP (ناشناس‌شده) / user / endpoint.

---

## ۶. Workers

### ۶.۱. AI Worker (`apps/worker-ai`)

- کار:
  - مصرف Queue `feedback:ai`,
  - Prompt سقراطی،
  - تماس با LLM،
  - validate ساختار JSON خروجی،
  - ذخیرهٔ `ai_feedback`,
  - update `submissions.status`.

- نکات:
  - در صورت پاسخ نامعتبر از LLM:
    - Retry محدود (مثلاً ۳ بار با Backoff نمایی)،
    - در صورت شکست نهایی، علامت‌گذاری برای مداخلهٔ انسانی.
  - Logging کامل برای Debug و ارسال Errorها به Sentry.

### ۶.۲. Publishing Worker (`apps/worker-publisher`)

- کار:
  - واترمارک EPUB نسخهٔ دیجیتال کتاب،
  - صدور DownloadToken،
  - (در آینده) تولید Anthology (گزیدهٔ ماه/فصل).

- Flow واترمارک:
  1. دریافت `ebookId` و شناسهٔ کاربر (یا ایمیل).
  2. خواندن EPUB خام از S3.
  3. محاسبۀ hash ایمیل کاربر (مثلاً SHA-256),
  4. درج hash در Metadata/CSS،
  5. ذخیرهٔ فایل واترمارک‌شده در Bucket `ebooks`.
  6. ایجاد/به‌روزرسانی `download_tokens` و اتصال آن به user.

### ۶.۳. Crypto/NFT Worker (`apps/worker-crypto`)

- کار در MVP:
  - پایش تراکنش USDT روی Polygon (برای کاربران خارج از ایران)،
  - Verify مبلغ و مقصد،
  - ثبت در DB (`crypto_payments`),
  - اتصال تراکنش به محصول (نسخۀ دیجیتال کتاب، اشتراک، اعتبار نقد انسانی) و فعال‌سازی آن‌ها،
  - Mint NFT برای نسخۀ دیجیتال کتاب و ثبت در `wallet_token_links`.

- در فازهای بعد:
  - مدیریت سناریوهای پیشرفته‌تر (مثل Gift، انتقال NFT، Integrations با بازارهای ثانویه) بدون تعهد به قیمت/سود.

---

## ۷. Edge & Security

### ۷.۱. Cloudflare CDN & WAF

- Delivery:
  - استاتیک‌ها و صفحات SSG/ISR از Edge نزدیک به کاربر تحویل داده می‌شوند.
- WAF:
  - قواعد پایه برای حملات معمول (XSS, SQLi, …).
  - Rate Limits پایه برای IPها/کشورها در صورت نیاز.

### ۷.۲. IP Anonymizer Worker

- خواندن `CF-Connecting-IP`,
- تبدیل IP کامل به `x.y.z.0`,
- افزودن `X-Anonymized-IP` به درخواست,
- Backend و لاگ‌ها فقط همین IP ماسک‌شده را ذخیره و تحلیل می‌کنند.

### ۷.۳. منطق Edge فصل شانزدهم (آینده)

- مدیریت Fingerprint و Rate Limiting واکنش‌های بدون حساب کاربری می‌تواند به Edge منتقل شود تا:
  - فشار روی Origin کمتر شود،
  - حریم خصوصی بیشتر رعایت شود (کمینه‌سازی داده‌های منتقل‌شده).

---

## ۸. عدم اهداف (Non-Goals)

- **Microservices سنگین (Kafka, K8s, Database per Service):**
  - برای این پروژه، به‌طور آگاهانه رد شده‌اند؛  
    چون:
    - پیچیدگی داده و حذف حساب را دشوار می‌کنند،
    - نسبت به مقیاس و بودجۀ ۶ ماهه، overkill هستند.

- **Headless CMS خارجی برای محتوای اصلی کتاب/تمرین‌ها:**
  - Content بخش اصلی دامنه است و باید در Postgres و Context Content مدیریت شود؛
  - استفاده از CMS خارجی فقط برای بخش‌های غیرهسته‌ای (وبلاگ، اخبار…) در آینده قابل بررسی است.

---

## ۹. جمع‌بندی

معماری Fortified Monolith به شکل نهایی زیر تثبیت می‌شود:

- Backend یکپارچه، دامنه‌محور، با Contextهای واضح،  
- Workerهای جدا برای AI/Crypto/Publishing،  
- ContentItem/ContentLinks برای حل محتوای عمومی/تکمیلی پیش و پس از کتاب،  
- AI سقراطی، با ساختار JSON مشخص و نقش مربی نه مصحح،  
- دو‌زبانگی در معماری از روز اول، با rollout محتوای EN تدریجی،  
- Book Mode کنترل‌شده با Feature Flag تا زمان چاپ،  
- Edge/CDN قدرتمند برای رسیدن به اهداف عملکردی در شرایط اینترنت ایران (مطابق [فصل نوزدهم سند دامنه](./product-domain-spec.md#chapter-19)),  
- Observability مبتنی بر Prometheus/Grafana/Loki/Sentry،  
- معماری منطبق با سیاست‌های حریم خصوصی و حذف/ناشناس‌سازی داده‌های جامعه‌محور (مطابق [فصل هفدهم و هجدهم](./product-domain-spec.md#chapter-17)),  
- پیاده‌سازی کامل فیچر «حلقه‌های نقد (Critique Circles)» طبق مشخصات `feature-spec-critique-circles.md` و بریف فنی، شامل جداسازی حلقه‌های نوجوانان و بزرگسالان،  
- پشتیبانی از زمین بازی ایده‌ها (Idea Playground) و مود Creative Clash به‌عنوان فیچر خلاقیت سبک در MVP،  
- و پیاده‌سازی Commerce & Crypto به‌گونه‌ای که برای کاربران داخل ایران از درگاه ریالی و برای کاربران خارج از ایران از USDT/Polygon (و در مورد کتاب دیجیتال، EPUB+NFT) استفاده شود، بدون ورود به حوزهٔ مشاورهٔ سرمایه‌گذاری.

این سند، مرجع نهایی برای تمام تصمیمات معماری است. هر تغییر بعدی در معماری باید در قالب ADR جداگانه مستند و با این سند تطبیق داده شود.
```

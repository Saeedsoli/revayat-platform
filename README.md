
# پلتفرم آموزش داستان‌نویسی با رویکرد ترکیبی  
## Fortified Monolith – Monorepo

این مخزن، پیاده‌سازی فنی پلتفرم آموزش داستان‌نویسی با رویکرد ترکیبی (پلتفرم‌اول + کتاب چاپی ۱۶ فصلی) است.  
تمام طراحی دامنه، معماری و برنامهٔ ساخت در اسناد زیر تعریف شده است:

- `product-domain-spec.md` – سند دامنهٔ ۳۳ فصلی (قلب پروژه)
- `architecture-fortified-monolith.md` – معماری فنی Fortified Monolith
- `technical-brief.md` – بریف فنی MVP + فاز ۱ خلاقیت
- `build-plan-stage5.md` – برنامهٔ ساخت مرحلهٔ ۵ (MVP + Critique Circles + خلاقیت)
- `feature-spec-critique-circles.md` – مشخصات کامل فیچر Critique Circles

این README، راهنمای کلی برای توسعه‌دهندگان و مشارکت‌کنندگان است:  
از معماری و ساختار Monorepo تا راه‌اندازی محیط توسعه، تنظیمات محیطی، اجرا، تست و استقرار.

---

## ۱. معرفی محصول در یک نگاه

### ۱.۱. محصول چیست؟

یک پلتفرم آموزش داستان‌نویسی که:

- یک **کتاب چاپی ۱۶ فصلی** دارد (۱۵ فصل متنی + فصل شانزدهم ساختارشکن با کد پاسخ‌گو)،
- یک **پلتفرم آنلاین تمرین و بازخورد** دارد که:
  - مسیرهای آموزشی عملی ارائه می‌کند («از ایده تا طرح کوتاه» در MVP)،
  - بازخورد ترکیبی (AI سقراطی + منتور انسانی) می‌دهد،
  - حلقه‌های نقد همتا (Critique Circles) دارد،
  - مرکز محتوای تکمیلی (ContentItems) و Idea Playground برای بازی خلاقیت فراهم می‌کند.

**اصل کلیدی:**  
اول پلتفرم، بعد کتاب؛ کتاب بر اساس دادهٔ واقعی و تمرین‌های بهینه‌شده چاپ می‌شود، نه صرفاً بر اساس تئوری.

### ۱.۲. چه چیزی MVP را خاص می‌کند؟

- **Critique Circles** – حلقه‌های نقد همتا، Exercise-specific، با جداسازی Teen/Adult و قاعدهٔ «اول بده، بعد بگیر» با Participation Lock پویا.
- **AI سقراطی** – بازخورد غیرنسخه‌نویس، پرسش‌محور و توصیفی (نه نمرهٔ خشک).
- **Idea Playground / Creative Clash** – مود خلاقیت سبک برای گرم‌کردن ذهن، بدون نمره یا فشار.
- **حریم خصوصی جدی** – IP ناشناس‌شده (IPv4 و IPv6)، Analytics اختیاری (Opt-in)، حذف حساب دوحالته (ناشناس‌سازی/حذف کامل) و استفادهٔ محدود از سرویس‌های AI بیرونی (Data Processor، بدون Training عمومی).

جزئیات در:

- `product-domain-spec.md` (فصل ۱–۶، ۸، ۱۴، ۱۷)
- `technical-brief.md` (بخش‌های ۱ و ۴)

---

## ۲. معماری کلان

### ۲.۱. Fortified Monolith

- **Backend:** یک Monolith ماژولار Node.js/TypeScript با Contextهای دامنه‌ای:
  - Identity & Access
  - Learning & Practice
  - Feedback (AI & Human)
  - Content & Chapters & ContentItems
  - Community & Featured (Critique Circles, Featured Samples)
  - Commerce & Billing
  - Compliance & Data Governance
  - Analytics & Events

- **Workers ایزوله:**
  - `worker-ai` – پردازش بازخورد AI سقراطی روی Queue (`feedback:ai`)
  - `worker-publisher` – واترمارک EPUB و صدور DownloadToken
  - `worker-crypto` – تأیید پرداخت USDT/Polygon و Mint NFT

- **Edge:**
  - Cloudflare (CDN + WAF + Workers) برای:
    - کش/تحویل HTML و استاتیک‌ها،
    - ناشناس‌سازی IP (IPv4 و IPv6) و ارسال در `X-Anonymized-IP`,
    - Rate Limiting برخی Endpointها (Auth، Submit، Critique APIs، واکنش‌ها).

### ۲.۲. پشتهٔ فناوری (نسخه‌های مرجع)

- Frontend:  
  - Next.js 16.0.4 (App Router)  
  - React 19.2  
  - TypeScript (strict)  
  - Design System RTL-ready (Tailwind یا معادل)

- Backend:  
  - Node.js 24.11.1 (LTS)  
  - Fastify یا Express (Monolith)  
  - TypeScript (strict)

- Data & Infra:  
  - PostgreSQL 18.1 (منبع حقیقت)  
  - Valkey 9.0.0 (Cache, Queue, Rate Limit)  
  - Meilisearch 1.27.0 (جست‌وجوی تمام‌متن)  
  - Object Storage سازگار با S3 (Cloudflare R2/MinIO)

- Auth & Community:  
  - Supabase Auth (Email+Password, Discord)  
  - Discord Bot (نقش‌دهی، اعلان‌ها، اعمال محدودیت سنی در سرور دیسکورد)

- Observability:  
  - Prometheus + Grafana + Loki + Sentry  
  - در لاگ‌ها و متریک‌ها، فقط IP ناشناس‌شده ذخیره می‌شود.

جزئیات کامل در:  
`architecture-fortified-monolith.md` (بخش ۲ و ۳)

---

## ۳. ساختار Monorepo

ساختار پیشنهادی (و مورد انتظار اسناد):

```text
.
├─ apps/
│  ├─ web/             # Next.js (Public + App + Admin + Playground)
│  ├─ api/             # Backend Monolith (Node.js/TS)
│  ├─ worker-ai/       # Worker AI Feedback
│  ├─ worker-crypto/   # Worker Crypto/NFT
│  └─ worker-publisher/# Worker Publishing (EPUB/DownloadToken/Anthology)
│
├─ packages/
│  ├─ database/        # اسکیما و کلاینت DB (Prisma/Drizzle)
│  └─ shared/          # Types، DTOها، Zod Schemas، utilities دامنه‌ای مشترک
│
├─ docker/
│  ├─ docker-compose.dev.yml        # Postgres, Valkey, Meilisearch, MinIO
│  └─ docker-compose.monitoring.yml # (اختیاری) Prometheus/Grafana/Loki
│
├─ product-domain-spec.md
├─ architecture-fortified-monolith.md
├─ technical-brief.md
├─ build-plan-stage5.md
├─ feature-spec-critique-circles.md
└─ ...
```

ابزار مدیریت مخزن:

- **Turborepo** (در `turbo.json`)  
- **pnpm** (با `pnpm-workspace.yaml`)  
- **tsconfig.base.json** برای اشتراک تنظیمات TS

---

## ۴. پیش‌نیازها

برای توسعه و اجرای محلی:

- **Node.js** 24.x (ترجیحاً 24.11.1 LTS مطابق سند نسخه‌ها)
- **pnpm** (آخرین نسخهٔ پایدار)
- **Docker & Docker Compose** (برای DB، Redis/Valkey، Meilisearch، MinIO)
- **Git**
- یک حساب:
  - **Supabase** (برای Auth)،
  - **Cloudflare** (برای DNS/CDN/WAF/Workers)،
  - **Discord** (برای Bot و احراز هویت اختیاری)،
  - در فاز بعد: Node Provider برای Polygon (Crypto Worker).

---

## ۵. تنظیمات محیطی (Environment)

### ۵.۱. متغیرهای پایه (Root / Backend)

نمونه‌ای از `.env` (برای توسعه – مقادیر نمونه/فرضی):

```env
# Backend API
NODE_ENV=development
PORT=4000

# Database
DATABASE_URL=postgres://user:pass@localhost:5432/storyplatform

# Valkey (Redis-compatible)
VALKEY_URL=redis://localhost:6379

# Meilisearch
MEILISEARCH_HOST=http://localhost:7700
MEILISEARCH_API_KEY=masterKey

# Supabase
SUPABASE_URL=https://your-supabase-instance.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
SUPABASE_ANON_KEY=your-anon-key

# Auth & JWT (در صورت نیاز)
JWT_SECRET=change-me

# Cloudflare (در محیط‌های بالاتر)
CLOUDFLARE_ACCOUNT_ID=...
CLOUDFLARE_API_TOKEN=...

# Critique Circles
CRITIQUE_REQUIRED_REVIEWS=2

# Observability
SENTRY_DSN_BACKEND=...
PROMETHEUS_METRICS_PORT=9100
```

### ۵.۲. Frontend (`apps/web`)

```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:4000
NEXT_PUBLIC_ENABLE_BOOK_MODE=false
NEXT_PUBLIC_SUPABASE_URL=https://your-supabase-instance.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

### ۵.۳. Workers

هر Worker معمولاً از همان `DATABASE_URL`, `VALKEY_URL`, ... استفاده می‌کند + متغیرهای مخصوص خودش (مثل API Key سرویس LLM، Node Provider برای Polygon، و ...).  
جزئیات کامل در `architecture-fortified-monolith.md` و `build-plan-stage5.md` آمده است.

---

## ۶. راه‌اندازی محیط توسعه

### ۶.۱. گام‌به‌گام

۱. **کلون مخزن:**

```bash
git clone <REPO_URL> story-platform
cd story-platform
```

۲. **نصب وابستگی‌ها (با pnpm):**

```bash
pnpm install
```

۳. **راه‌اندازی سرویس‌های زیرساختی (Postgres, Valkey, Meilisearch, MinIO):**

```bash
docker compose -f docker/docker-compose.dev.yml up -d
```

۴. **اجرای Migrationها و Seedهای پایه:**

- اگر از Prisma استفاده می‌شود:

```bash
cd packages/database
pnpm prisma migrate dev
pnpm prisma db seed
```

- اگر از Drizzle استفاده می‌شود، معادل آن (`drizzle-kit push` و ...).

۵. **اجرای Backend و Frontend در حالت توسعه:**

در یک ترمینال:

```bash
pnpm dev --filter api
```

در ترمینال دیگر:

```bash
pnpm dev --filter web
```

یا اگر اسکریپت `dev` در root برای اجرای هم‌زمان تعریف شده:

```bash
pnpm dev
```

۶. **اجرای Workers (در صورت نیاز در توسعه):**

AI Worker:

```bash
pnpm dev --filter worker-ai
```

Crypto Worker و Publishing Worker نیز مشابه.

> طراحی دقیق اسکریپت‌ها در `package.json` باید مطابق این الگو باشد؛ در صورت اختلاف، این README باید با کد هم‌سو شود.

---

## ۷. لایه‌های دامنه و ماژول‌ها

### ۷.۱. Identity & Access

- ادغام Supabase Auth برای:
  - ثبت‌نام با ایمیل/رمز،
  - ورود با ایمیل/رمز،
  - ورود با Discord OAuth.
- جدول `users` شامل:
  - `auth_user_id` (پیوند ۱:۱ با Supabase)،
  - ایمیل، نام مستعار، تاریخ تولد، کشور، زبان ترجیحی، جنسیت و فیلدهای اختیاری.
- Middleware `auth` و `rbac` برای کنترل نقش‌ها.

**نقطهٔ مهم:**  
برای Users ایجادشده از طریق Discord، بعد از OAuth یک صفحهٔ تکمیل پروفایل نمایش داده می‌شود تا فیلدهای ضروری دامنه (dob, country, preferred_language, gender, display_name) گرفته شود.

### ۷.۲. Learning & Practice

- مسیر «از ایده تا طرح کوتاه» در DB (`learning_paths`, `sessions`, `exercises`) تعریف می‌شود.
- برای هر تمرین:
  - `start` → ایجاد Submission با `status='draft'`.
  - Autosave → `PATCH /submissions/{id}`.
  - `submit` → تغییر `status='submitted'`، اعمال محدودیت ۱۰ Submit نهایی/روز، ارسال Job برای AI Worker.

### ۷.۳. Feedback (AI & Human)

- AI Worker:
  - صف `feedback:ai` را مصرف می‌کند،
  - Prompt سقراطی می‌سازد،
  - LLM را صدا می‌زند،
  - JSON خروجی را Validate و در `ai_feedback` ذخیره می‌کند.
- Human Feedback:
  - منتورها از پنل `/admin/mentors` صف Submissionها را می‌بینند،
  - با `POST /mentor/submissions/{id}/feedback` نقد می‌نویسند.

Triggerهای ارجاع به منتور:

- خرابی مکرر AI،
- پرچم کاربر («این بازخورد مناسب نبود»)،
- محتوای حساس.

### ۷.۴. Community & Critique Circles

- حلقه‌های نقد همتا مطابق `feature-spec-critique-circles.md`:
  - Exercise-specific،
  - جداسازی Teen/Adult،
  - ظرفیت پیش‌فرض ۳ (Config ۲–۱۰)،
  - «حلقۀ فعال برای کاربر» = عضویت + `peer_feedback_unlocked=false` برای Submission خود او،
  - Participation Lock پویا (N=min(2, تعداد تمرین‌های دیگران در حلقه برای او))،
  - حداقل طول نقد ۲۰۰ و حداکثر ۵۰۰۰ کاراکتر،
  - self-review ممنوع و فقط یک نقد per (circle, submission, reviewer).

APIهای اصلی:

- `POST /api/circles/join`
- `GET /api/circles/my`
- `POST /api/peer-feedback`
- `POST /api/circles/report`

### ۷.۵. Commerce & Billing

- داخل ایران (`country='IR'`):
  - زرین‌پال برای کتاب چاپی، EPUB، اشتراک و اعتبار نقد انسانی.
- خارج از ایران (`country!='IR'`):
  - USDT/Polygon برای EPUB+NFT، اشتراک‌ها، اعتبار نقد انسانی؛
  - Crypto Worker تراکنش را Verify کرده و Publishing Worker و Mint NFT را اجرا می‌کند.

### ۷.۶. Compliance & Data Governance

- `AccountDeletionService`:
  - دو حالت:
    - ناشناس‌سازی مشارکت‌ها (حذف User و Set NULL کردن FKها در جداول جامعه‌محور)،
    - حذف کامل (حذف Submissionها و نقدها و وابستگی‌ها).
  - اجرای هم‌زمان در MVP (کاربر در چند ثانیه نتیجه را می‌بیند).
- `reports`, `security_events` و Retention ۳۰/۹۰/۷سال مطابق سند دامنه.

### ۷.۷. Analytics & Skill Tree

- `events_behavior` (Opt-in برای کاربران لاگین‌شده؛ برای مهمان‌ها بدون user_id)، نگه‌داری ۳۰ روز.
- `events_technical`/`security_events`، نگه‌داری ۹۰ روز.
- Aggregationهای غیرشخصی (Skill Tree) بلندمدت.
- Skill Tree:
  - فقط آمار تجمیعی ساده (تعداد تمرین‌ها، مسیرها، کاربران فعال)،
  - بدون Leaderboard یا امتیاز فردی عمومی،
  - نمودار رشد فردی فقط برای خود کاربر.

---

## ۸. تست‌ها

### ۸.۱. انواع تست‌ها

- **تست‌های واحد (Unit Tests):**
  - Services، Repos و Components کلیدی.
- **تست‌های Integration:**
  - مسیر ثبت‌نام → تمرین → AI Feedback → Critique Circles → حذف حساب.
- **تست‌های End-to-End (در صورت امکان):**
  - با Playwright/Cypress، برای سناریوهای اصلی کاربر.

### ۸.۲. سناریوهای کلیدی برای Critique Circles

حتماً تست شوند (مطابق `build-plan-stage5.md`):

1. کاربر عادی: تنها می‌تواند در یک حلقۀ فعال باشد.
2. کاربر دارای اشتراک: یک حلقۀ فعال در هر Path؛ در سومین حلقه هشدار دریافت می‌کند.
3. Teen فقط در حلقه‌های Teen؛ Adult فقط در حلقه‌های Adult.
4. Participation Lock:
   - در حلقۀ سه‌نفره: unlock بعد از دو نقد معتبر روی دو تمرین دیگران،
   - در حلقۀ دو‌نفره: unlock بعد از یک نقد معتبر (N_effective=1).

---

## ۹. استقرار (Deployment)

### ۹.۱. ایمیج‌های Docker

برای هر اپ:

- `apps/web`  
- `apps/api`  
- `apps/worker-ai`  
- `apps/worker-publisher`  
- `apps/worker-crypto`  

ایمیج‌های جداگانه ساخته می‌شود. مثال:

```bash
docker build -t story-web ./apps/web
docker build -t story-api ./apps/api
docker build -t story-worker-ai ./apps/worker-ai
...
```

### ۹.۲. محیط‌ها

- **Staging:**
  - برای تست یکپارچۀ قبل از لانچ، با Config نزدیک به Production.
- **Production:**
  - روی یکی از بسترهای ابری (Hetzner/AWS/لیارا/ابرآروان) مطابق سند معماری.

### ۹.۳. Edge و DNS

- تنظیم دامنه در Cloudflare،
- استقرار Worker IP Anonymizer،
- فعال‌سازی CDN و WAF،
- Rate Limiting روی مسیرهای حساس.

---

## ۱۰. استانداردهای مشارکت (Contribution Guidelines)

۱. **قبل از هر تغییر معنی‌دار دامنه/معماری:**
   - ابتدا در یکی از اسناد زیر Spec/Proposal را بنویسید:
     - `product-domain-spec.md` – برای تغییرهای دامنه‌ای،
     - `architecture-fortified-monolith.md` – برای تغییرهای معماری،
     - `feature-spec-*.md` – برای فیچرهای جدید.
   - سپس ADR جداگانه ایجاد کنید.

2. **Pull Requestها:**
   - باید:
     - تست‌های مربوطه را اضافه/به‌روزرسانی کنند،
     - اسناد مرتبط را به‌روز کنند (Domain, Architecture, Technical Brief, Feature Spec, Build Plan).

3. **استفاده از Feature Flags:**
   - برای فیچرهای جدید (مثلاً مودهای جدید در Idea Playground) حتماً پشت Feature Flag پیاده‌سازی شوند.

4. **رعایت حریم خصوصی:**
   - هر تغییر در مدل داده، مخصوصاً اگر دادهٔ جدیدی از کاربر بگیرد یا Retention را تغییر دهد، باید با [فصل هفدهم و هجدهم سند دامنه](./product-domain-spec.md#chapter-17) سازگار باشد.

---

## ۱۱. منابع تکمیلی

- **سند دامنه (۳۳ فصل):**  
  `product-domain-spec.md`  
  – تعریف دقیق دامنه، ارزش‌ها، سیاست داده، سنی، کسب‌وکار،…  

- **سند معماری:**  
  `architecture-fortified-monolith.md`  
  – الگوی Fortified Monolith، Contextها، Tech Stack، Edge، Workers، Observability.

- **بریف فنی:**  
  `technical-brief.md`  
  – تصویر فنی MVP، دامنهٔ فیچرها، Skill Tree، Idea Playground، و ارتباط دامنه/معماری.

- **Spec Critique Circles:**  
  `feature-spec-critique-circles.md`  
  – رفتار دقیق حلقه‌های نقد، مدل داده، API، UX، خطاها.

- **برنامهٔ ساخت مرحله ۵:**  
  `build-plan-stage5.md`  
  – فازبندی اجرا (Infra → DB → Backend → Workers → Frontend → Edge/Discord → Tests → Deploy).

---

این README نقطهٔ شروع برای توسعه‌دهندگان است.  
برای هر تغییری که می‌خواهید انجام دهید، ابتدا سند مناسب را بخوانید و مطمئن شوید که:

1. با Domain Spec و ارزش‌های پروژه هم‌خوان است؛  
2. با Architecture و Technical Brief سازگار است؛  
3. در صورت لزوم، در قالب ADR مستند شده است.

با این کار، کل تیم (و آیندۀ پلتفرم) از «سر خوردن مفاهیم» و ناسازگاری‌های خطرناک در امان می‌ماند.
```

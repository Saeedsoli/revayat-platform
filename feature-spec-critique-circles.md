<!-- File: feature-spec-critique-circles.md -->

# مشخصات فنی کامل فیچر Critique Circles (حلقه‌های نقد همتا)

**نسخه:** ۱.۱.۰  
**تاریخ مرجع:** چهارشنبه، ۱۰ دسامبر ۲۰۲۵  
**وضعیت:** نهایی برای اجرا  
**معماری مرجع:** `architecture-fortified-monolith.md`  
**بریف مرجع:** `technical-brief.md`  
**سند دامنه مرجع:** `product-domain-spec.md` (به‌ویژه [فصل هشتم](./product-domain-spec.md#chapter-8) و [فصل چهاردهم](./product-domain-spec.md#chapter-14))  
**Context دامنه‌ای:** Community & Featured  

این سند، تعریف دقیق و بدون ابهام Critique Circles را ارائه می‌کند؛  
هر پیاده‌سازی Backend، Frontend، QA و DevOps باید با این مشخصات منطبق باشد.  
هر تغییری در رفتار این فیچر نیازمند ADR مستقل است.

> **نام‌گذاری:**  
> - در اسناد دامنه/UX: «حلقهٔ نقد (Critique Circle)» / «حلقه‌های نقد (Critique Circles)»  
> - در اسناد فنی و کد: **Critique Circles** به‌عنوان نام رسمی فیچر.

---

## ۱. هدف دامنه‌ای و نقش محصولی

### ۱.۱. تعریف دامنه‌ای

Critique Circles (حلقه‌های نقد) گروه‌های کوچک و موقتی از کاربران (در MVP معمولاً **سه‌نفره**) هستند که:

- در یک بازهٔ زمانی، تمرین‌های همدیگر را می‌خوانند،
- بر اساس فرم و چارچوب استاندارد، برای هم «نقد همتا» (Peer Feedback) می‌نویسند،
- و تنها پس از نقد دادن به دیگران، نقدهای دریافتی خود را مشاهده می‌کنند.

هر حلقه:

- برای **یک تمرین مشخص** (`exercise_id`) در یک مسیر آموزشی (`path_id`)،
- در یک زبان مشخص (`language`)،
- و در یک بازۀ سنی مشخص (`age_band`)  
تشکیل می‌شود. این فیچر، پیاده‌سازی فنی بخش «حلقه‌های نقد درون‌پلتفرمی (Critique Circles)» در [فصل هشتم سند دامنه](./product-domain-spec.md#chapter-8) است و در Context دامنه‌ای **Community & Featured** جای می‌گیرد.

### ۱.۲. نقش این فیچر در محصول

- افزودن یک **لایهٔ یادگیری همتا به همتا (Peer Learning)** در کنار AI و منتور.
- ایجاد حس «جامعهٔ کوچک امن» در کنار زیست‌بوم بزرگ‌تری مثل دیسکورد.
- ایجاد انگیزه برای مشارکت سازنده، نه فقط مصرف بازخورد.
- فراهم‌کردن محیطی امن‌تر برای نوجوانان با جداسازی حلقه‌های Teen (۱۳–۱۷) و Adult (۱۸+).

### ۱.۳. هم‌خوانی با ارزش‌ها (ارجاع به فصل یکم دامنه)

- **آزادی اندیشه و عمل:**  
  چند دیدگاه هم‌سطح به جای یک «صدای یگانهٔ درست»؛ نقدها پیشنهاد‌محور و غیرنسخه‌نویس‌اند.
- **احترام به کاربر:**  
  هر عضو هم دریافت‌کننده و هم دهندهٔ بازخورد است؛ سلسله‌مراتب سخت و تحقیرآمیز وجود ندارد.
- **شفافیت و عدالت:**  
  قاعدهٔ «اول بده، بعد بگیر» واضح است؛ هیچ‌کس بدون مشارکت، بازخورد رایگان نمی‌گیرد.
- **حریم خصوصی و اختیار:**  
  در زمان حذف حساب، کاربر می‌تواند انتخاب کند که نقدها و تمرین‌هایش ناشناس باقی بماند یا کاملاً حذف شود (مطابق [فصل هفدهم دامنه](./product-domain-spec.md#chapter-17)).  
  در حالت ناشناس‌سازی، پیوند فنی بین مشارکت‌ها و هویت کاربر از بین می‌رود (FKها به `NULL` ست می‌شوند).

---

## ۲. قواعد کسب‌وکاری (Business Rules)

این بخش قواعد رفتاری و محدودیت‌های Critique Circles را از دید دامنه توضیح می‌دهد. پیاده‌سازی فنی (DB/API/UI) باید دقیقاً منعکس‌کنندهٔ همین قواعد باشد.

۱. **حداقل سن:**
   - تنها کاربرانی که **حداقل ۱۳ سال** دارند می‌توانند حساب کاربری ایجاد کنند و در Critique Circles حضور داشته باشند.
   - کاربران زیر ۱۳ سال حساب کاربری ندارند و در Critique Circles شرکت نمی‌کنند.

۲. **ظرفیت حلقه:**
   - در MVP، ظرفیت پیش‌فرض هر حلقه `capacity = 3` است.
   - این عدد به‌صورت تنظیم سیستمی (Config) قابل‌تغییر است، اما تمام طراحی UI/UX بر مبنای حلقه‌های سه‌نفره است.
   - قیود منطقی:
     - `capacity` حداقل ۲ و حداکثر ۱۰؛ هر مقدار خارج از این بازه باید در لایۀ اپ جلوگیری شود.

3. **زبان حلقه (Language-specific):**
   - هر حلقه فقط یک زبان دارد (`'fa'` یا `'en'`).
   - حلقۀ فارسی فقط شامل کاربران/تمرین‌های با `language='fa'` است.
   - حلقۀ انگلیسی فقط شامل `language='en'` است.
   - Matching همیشه بر اساس `submission.language` انجام می‌شود.

4. **ارتباط با مسیر آموزشی و تمرین (Path-specific و Exercise-specific):**
   - هر حلقه به یک مسیر آموزشی (`learning_paths`) و یک تمرین مشخص (`exercises`) متصل است:
     - `path_id` و `exercise_id` هر حلقه در دیتابیس نگه‌داری می‌شود.
   - تنها Submissionهایی می‌توانند در یک حلقه باشند که:
     - متعلق به همان `exercise_id` و همان `path_id` و همان `language` باشند.
   - در MVP:
     - برای **هر تمرینی که Critique برای آن فعال شده (`is_critique_enabled=true`)**، Submissionهای کاربران در حلقه‌های جداگانه سازمان‌دهی می‌شوند؛  
       یعنی Critique Circles به‌طور کامل **Exercise-specific** هستند.
   - هر کاربر در هر حلقه حداکثر **یک Submission** دارد.

5. **بازه‌های سنی حلقه‌ها (Age-band-specific):**
   - دو بازۀ سنی تعریف می‌شود:
     - `teen`: سن ۱۳ تا ۱۷ سال (نوجوان)،
     - `adult`: سن ۱۸ سال به بالا (بزرگسال).
   - هر حلقه دقیقاً یک `age_band` دارد (`'teen'` یا `'adult'`).
   - Matching حلقه و کاربر:
     - کاربران teen (۱۳–۱۷) فقط می‌توانند در حلقه‌های teen حضور داشته باشند؛
     - کاربران adult (۱۸+) فقط در حلقه‌های adult عضو می‌شوند.
   - هیچ حلقۀ مختلط Teen+Adult وجود ندارد.

6. **عضویت در هر حلقه:**
   - هر کاربر برای هر تمرین مشخص، تنها در یک حلقه عضو می‌شود.
   - اگر کاربر دوباره برای همان Submission درخواست Join بدهد، رفتار باید idempotent باشد (همان حلقه برگردد، خطا ندهیم).
   - کاربر نمی‌تواند دو Submission متفاوت از خودش را در یک حلقه ثبت کند.

7. **تعریف «حلقۀ فعال برای کاربر» (Active Circle For User):**
   - برای اعمال محدودیت تعداد حلقه‌ها، «حلقۀ فعال برای کاربر» چنین تعریف می‌شود:
     - حلقه‌ای که کاربر در آن عضو است (`circle_members.left_at IS NULL`)،
     - و برای Submission خودش در آن حلقه، `peer_feedback_unlocked = false` است؛
       یعنی هنوز تعداد نقدهای لازم را برای دیگران ننوشته و نقدهای دیگران روی تمرین خودش unlock نشده است.
   - به‌محض این‌که کاربر حداقل تعداد نقد لازم را بنویسد و `peer_feedback_unlocked` روی Submission خودش در آن حلقه `true` شود، آن حلقه از نظر محدودیت سهمیه‌ای دیگر «فعال» برای او به حساب نمی‌آید، حتی اگر وضعیت حلقه در جدول `critique_circles.status` همچنان `active` باشد.

8. **محدودیت تعداد حلقه‌های فعال (در کل سیستم):**

   - **کاربر عادی (بدون اشتراک فعال):**
     - در هر لحظه حداکثر **یک حلقۀ فعال** (مطابق تعریف بند ۷) می‌تواند داشته باشد؛
     - اگر کاربر عادی تلاش کند به حلقۀ دومی بپیوندد، در حالی که هنوز در حلقۀ قبلی `peer_feedback_unlocked=false` دارد، درخواست رد می‌شود و پیام شفافی دریافت می‌کند که ابتدا باید حلقۀ فعلی خود را تکمیل کند.

   - **کاربر دارای اشتراک فعال:**
     - «کاربر دارای اشتراک فعال» یعنی:
       - کاربری که حداقل یک رکورد در جدول `subscriptions` دارد با:
         - `status = 'active'`
         - `expires_at > NOW()`
       - صرف خرید کتاب چاپی یا دیجیتال او را در این دسته قرار نمی‌دهد (مطابق [فصل دهم دامنه](./product-domain-spec.md#chapter-10)).
     - این کاربر می‌تواند در **هر مسیر آموزشی (Path) حداکثر یک حلقۀ فعال** داشته باشد (با تعریف فوق از حلقۀ فعال برای او).
     - اگر یک کاربر دارای اشتراک فعال در دو حلقۀ فعال (در دو Path مختلف) عضو باشد و برای سومی درخواست Join بدهد:
       - Join از نظر کسب‌وکار مجاز است؛
       - اما Backend باید در پاسخ، فلگ هشدار (مثلاً `warning: true, warningCode: 'MULTIPLE_ACTIVE_CIRCLES'`) برگرداند تا UI یک بنر غیرمسدودکننده نشان دهد («شما در ۳ حلقۀ هم‌زمان فعال هستید»).

9. **وضعیت حلقه (`status`):**
   - مقدارهای مجاز:
     - `open` – حلقه هنوز کامل نشده است (تعداد اعضا < capacity)؛ اعضای جدید می‌توانند بپیوندند.
     - `active` – حلقه به ظرفیت کامل خود رسیده و آمادهٔ نقد متقابل است.
     - `closed` – برای آینده رزرو؛ در MVP به صورت خودکار استفاده نمی‌شود.
   - در MVP:
     - وقتی تعداد اعضا به ظرفیت برسد → `status='active'`.
     - حتی پس از پایان نقدها توسط همهٔ اعضا، `status` همچنان `active` باقی می‌ماند؛  
       بستن رسمی حلقه (تغییر به `closed`) فقط از طریق ابزارهای Admin و در فازهای بعدی خواهد بود.
   - نکتهٔ مهم:
     - وضعیت `status` حلقه مستقیماً برای محدودیت‌های تعداد حلقهٔ کاربر استفاده نمی‌شود؛ برای این محدودیت، تعریف «حلقۀ فعال برای کاربر» در بند ۷ ملاک است.

10. **قفل مشارکت (Participation Lock) و N پویا:**
    - کاربر تا زمانی که حداقل **N نقد** (در MVP، N بر اساس قاعدۀ پویا تعیین می‌شود) برای تمرین‌های دیگر اعضای حلقه ننوشته است:
      - نقدهای دریافت‌شده روی تمرین خودش را نمی‌بیند (`peer_feedback_unlocked=false`).
    - در درجات دامنه‌ای، برای هر کاربر در هر حلقه، تعداد نقد لازم به این صورت تعریف می‌شود:

      > `N_effective = min(N_base, تعداد Submissionهای دیگران در همان حلقه برای این کاربر)`

      که در MVP مقدار `N_base` برابر **۲** است (به‌صورت Config مانند `CRITIQUE_REQUIRED_REVIEWS=2` نگه‌داری می‌شود).  
      یعنی:
      - در حلقۀ سه‌نفره (هر کاربر دو تمرین دیگران را می‌بیند)، `N_effective = 2`;  
      - در حلقۀ دو‌نفره (فقط یک تمرین متعلق به نفر دیگر است)، `N_effective = 1`.
    - در عمل:
      - نقدهایی شمرده می‌شوند که:
        - روی Submission دیگران نوشته شده باشند (self-review شمرده نمی‌شود)،
        - حداقل طول متنی را داشته باشند (بند بعد).
      - پس از ثبت دست‌کم `N_effective` نقد معتبر، نقدهای دریافتی روی تمرین کاربر unlock می‌شوند (`peer_feedback_unlocked=true`).

11. **حداقل و حداکثر طول نقد:**
    - متن نقد (`peer_feedback.body`) باید:
      - **حداقل ۲۰۰** کاراکتر،
      - و **حداکثر ۵۰۰۰** کاراکتر  
      باشد.
    - این قید در Backend enforce می‌شود؛ UI فقط راهنما نشان می‌دهد، اما Validation اصلی روی سرور است.

12. **سایر محدودیت‌ها:**
    - self-review ممنوع: کاربر نمی‌تواند روی تمرین خودش نقد بنویسد.
    - هر کاربر روی هر Submission در یک حلقه فقط یک نقد می‌تواند بنویسد.
    - تلاش برای نقد دوم روی همان Submission در همان حلقه → خطای `409 Conflict` با کد `DUPLICATE_FEEDBACK`.

13. **ارتباط با AI Feedback:**
    - پیوستن به حلقۀ نقد **وابسته به آماده‌شدن AI Feedback نیست**:
      - کاربر بلافاصله بعد از `submit` می‌تواند وارد حلقه شود.
      - AI Feedback مسیر خودش را موازی می‌رود و در صفحهٔ بازخورد تمرین نمایش داده می‌شود، اما Participation Lock حلقه به آن وابسته نیست.
    - این استقلال، تجربهٔ کاربر را از تأخیرهای احتمالی LLM جدا می‌کند (مطابق بریف و [فصل ششم دامنه](./product-domain-spec.md#chapter-6)).

14. **پایان حلقه در MVP:**
    - حلقه‌ها در MVP الزاماً به صورت خودکار `closed` نمی‌شوند.
    - از نظر دامنه‌ای، حلقه زمانی «عملیاتاً تمام‌شده» تلقی می‌شود که:
      - هر عضو حداقل `N_effective` نقد نوشته باشد،
      - و نقدهای دریافتی روی تمرین همهٔ اعضا unlock شده باشد.
    - در MVP:
      - همین که Unlock انجام شود، از دید کاربر حلقه مأموریتش را انجام داده،  
        اما `status` در DB همچنان `active` باقی می‌ماند؛  
        منطق بستن خودکار در فازهای بعدی طراحی خواهد شد.
      - این موضوع با تعریف «حلقۀ فعال برای کاربر» در بند ۷ سازگار است؛ پس از Unlock، حلقه دیگر در محدودیت‌ها برای او حساب نمی‌شود.

---

## ۳. مدل داده (Database Schema)

### ۳.۱. جدول `critique_circles`

نمایندهٔ یک حلقهٔ نقد.

```sql
critique_circles (
  id          UUID PRIMARY KEY,
  name        TEXT NULL,              -- نام نمایشی اختیاری
  status      TEXT NOT NULL,          -- 'open' | 'active' | 'closed'
  capacity    INTEGER NOT NULL,       -- پیش‌فرض: 3، بین 2 تا 10
  language    TEXT NOT NULL,          -- 'fa' | 'en'
  path_id     UUID NOT NULL REFERENCES learning_paths(id),
  exercise_id UUID NOT NULL REFERENCES exercises(id),
  age_band    TEXT NOT NULL,          -- 'teen' | 'adult'
  owner_id    UUID NULL REFERENCES users(id) ON DELETE RESTRICT,
  created_at  TIMESTAMPTZ NOT NULL,
  updated_at  TIMESTAMPTZ NOT NULL
);
```

قیود:

- `status` فقط `'open'`, `'active'`, `'closed'`.
- `capacity` ≥ ۲ و ≤ ۱۰ (MVP: ۳، Config).
- `language` فقط `'fa'` یا `'en'`.
- `age_band` فقط `'teen'` یا `'adult'`.

نکات:

- `owner_id` سازندهٔ حلقه را نگه می‌دارد؛ در MVP نقشی مدیریتی ندارد (بخش ۸ را ببینید).
- `ON DELETE RESTRICT` روی `owner_id` یعنی قبل از حذف کاربر، باید تصمیم دربارۀ حلقه‌های متعلق به او در Application گرفته شود (توسط `AccountDeletionService`).
- در منطق اپ:
  - هنگام ایجاد حلقه، `exercise_id`، `path_id`, `language` و `age_band` از Submission/کاربر گرفته می‌شود؛
  - مطابقت این فیلدها با تمرین‌ها و سِن اعضا در زمان Join enforce می‌شود.

### ۳.۲. جدول `circle_members`

عضویت کاربران در حلقه‌ها.

```sql
circle_members (
  id          UUID PRIMARY KEY,
  circle_id   UUID NOT NULL REFERENCES critique_circles(id) ON DELETE CASCADE,
  user_id     UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  role        TEXT NOT NULL,          -- 'member' | 'owner'
  joined_at   TIMESTAMPTZ NOT NULL,
  left_at     TIMESTAMPTZ NULL
);
```

قیود:

- `UNIQUE(circle_id, user_id)` – هر user فقط یک‌بار در یک حلقه عضو است.
- `role` فقط `'member'` یا `'owner'`.

نکات حریم خصوصی:

- FK `user_id → users(id)` دارای `ON DELETE RESTRICT` است:
  - حذف حساب کاربر باید ابتدا از طریق `AccountDeletionService` وابستگی‌ها را ناشناس/حذف کند (ست‌کردن به `NULL` یا حذف رکوردهای مربوطه طبق انتخاب کاربر)، سپس رکورد User حذف شود.

### ۳.۳. جدول `circle_submissions`

نگاشت Submissionهایی که موضوع نقد در یک حلقه هستند.

```sql
circle_submissions (
  id            UUID PRIMARY KEY,
  circle_id     UUID NOT NULL REFERENCES critique_circles(id) ON DELETE CASCADE,
  submission_id UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
  author_id     UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  created_at    TIMESTAMPTZ NOT NULL
);
```

قیود:

- `UNIQUE(circle_id, submission_id)` – هر Submission فقط یک‌بار در یک حلقه ثبت می‌شود.

قیود منطقی (در سطح اپ یا با Constraint/Trigger در آینده):

- `submissions.exercise_id` باید با `critique_circles.exercise_id` برابر باشد؛
- `submissions.language` و `critique_circles.language` باید هم‌خوان باشند؛
- `author_id` باید با `submissions.user_id` برابر باشد؛
- بازۀ سنی کاربر (`teen` یا `adult`) باید با `critique_circles.age_band` سازگار باشد.

نکات حریم خصوصی:

- FK `author_id → users(id)` با `ON DELETE RESTRICT`:
  - در حالت ناشناس‌سازی: این FK به `NULL` ست می‌شود؛  
  - در حالت حذف کامل مشارکت‌ها: رکورد `circle_submissions` حذف می‌شود.

### ۳.۴. جدول `peer_feedback`

نقد همتا روی یک Submission در یک حلقه.

```sql
peer_feedback (
  id            UUID PRIMARY KEY,
  circle_id     UUID NOT NULL REFERENCES critique_circles(id) ON DELETE CASCADE,
  submission_id UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
  reviewer_id   UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  body          TEXT NOT NULL,        -- متن نقد (حداقل 200، حداکثر 5000 کاراکتر)
  score         INTEGER NULL,         -- اختیاری، 1..5، اگر بخواهید
  created_at    TIMESTAMPTZ NOT NULL
);
```

قیود:

- `UNIQUE(circle_id, submission_id, reviewer_id)` – هر کاربر روی هر تمرین در یک حلقه یک نقد می‌نویسد.

قیود منطقی (در سطح اپ):

- `reviewer_id ≠ author_id` (self-review ممنوع).
- طول `body` باید در بازۀ [۲۰۰، ۵۰۰۰] کاراکتر باشد.

نکات حریم خصوصی:

- FK `reviewer_id → users(id)` با `ON DELETE RESTRICT`؛  
  در حالت ناشناس‌سازی، این FK به `NULL` ست می‌شود؛  
  در حالت حذف کامل، کل رکورد `peer_feedback` حذف می‌گردد (معمولاً همراه با حذف Submission در صورتِ انتخاب کاربر).

### ۳.۵. فیلدهای اضافی روی `submissions`

برای اتصال Submission به حلقه و unlock شدن نقدها:

```sql
ALTER TABLE submissions
  ADD COLUMN circle_id UUID NULL REFERENCES critique_circles(id) ON DELETE SET NULL,
  ADD COLUMN peer_feedback_unlocked BOOLEAN NOT NULL DEFAULT FALSE;
```

- در MVP، هر Submission در هر لحظه حداکثر یک `circle_id` دارد.
- `circle_id` یک Shortcut denormalized است؛ منبع اصلی عضویت `circle_submissions` است.
- **توضیح `ON DELETE SET NULL`:**  
  اگر یک حلقه حذف شود (مثلاً به صورت مدیریتی)، `circle_id` در Submissionهای مرتبط به `NULL` تبدیل می‌شود، بدون حذف Submission.

---

## ۴. APIها (قراردادهای HTTP)

تمام APIها JSON-based و نیازمند Auth هستند (user باید لاگین باشد).  
همه باید از middlewareهای `auth` و `rbac` استفاده کنند.

### ۴.۱. پیوستن به حلقه (Join Circle)

**URL:**  
`POST /api/circles/join`

**Auth:**  
- نیازمند کاربر Authenticated (با سن ≥ ۱۳).

**Body (JSON):**

```json
{
  "submissionId": "uuid"
}
```

> **توضیح:**  
> اطلاعات `pathId`, `exerciseId`, `language`, `age_band` از DB (Submission، Exercise و User) استخراج می‌شوند و از داده‌های Body خوانده نمی‌شوند.

**قواعد:**

1. `submissionId` الزامی است.
2. Submission باید:
   - متعلق به user فعلی باشد (`submissions.user_id = currentUserId`).
   - `status >= 'submitted'` داشته باشد (نه `draft`).
   - مربوط به تمرینی با `is_critique_enabled = true` باشد.
3. سن user باید ≥ ۱۳ باشد (این قاعده در لایۀ Identity enforce شده است).
4. تعیین `age_band` کاربر:
   - بر اساس `date_of_birth`:
     - سن ۱۳–۱۷ → `age_band='teen'`,
     - سن ≥ ۱۸ → `age_band='adult'`.
5. محدودیت تعداد حلقه:
   - منطبق با تعریف «حلقۀ فعال برای کاربر»:
     - اگر کاربر عادی و از قبل یک حلقۀ فعال دارد → خطا (`CIRCLE_LIMIT_REACHED`).
     - اگر کاربر دارای اشتراک فعال و در همان Path حلقۀ فعال دارد → خطا (`CIRCLE_LIMIT_REACHED_FOR_PATH`).
     - اگر کاربر دارای اشتراک فعال در دو حلقۀ فعال (در Pathهای مختلف) است و برای سومی Join می‌دهد:
       - Join مجاز است؛
       - Backend در پاسخ، فلگ `warning: true, warningCode: 'MULTIPLE_ACTIVE_CIRCLES'` برمی‌گرداند تا UI یک بنر غیرمسدودکننده نشان دهد.
6. اگر `submissions.circle_id` مقدار داشته باشد:
   - حلقۀ موجود واکشی و در پاسخ برگردانده می‌شود (idempotent).
7. اگر `submissions.circle_id` تهی باشد:
   - زبان و Path و Exercise و Age-band از DB:
     - `lang = submissions.language`,
     - `pathId = exercise.path_id`,
     - `exerciseId = exercise.id`,
     - `ageBand = userAgeBand`.
   - تلاش برای یافتن حلقۀ `open` با:
     - `language = lang`,
     - `path_id = pathId`,
     - `exercise_id = exerciseId`,
     - `age_band = ageBand`,
     - `status = 'open'`,
     - `membersCount < capacity`.
   - اگر حلقه یافت شد:
     - ایجاد/تثبیت `circle_members` (اگر user عضو نیست),
     - ایجاد `circle_submissions` (اگر برای این Submission هنوز ایجاد نشده),
     - به‌روزرسانی `submissions.circle_id`.
     - اگر ظرفیت پر شد → `status='active'`.
   - اگر یافت نشد:
     - حلقۀ جدید با:
       - `status='open'`,
       - `capacity=DEFAULT_CAPACITY (3)`,
       - `language=lang`,
       - `path_id=pathId`,
       - `exercise_id=exerciseId`,
       - `age_band=ageBand`,
       - `owner_id=currentUserId`;
     - سپس همان مراحل قبلی.

**پاسخ موفق (۲۰۰):**

```json
{
  "circle": {
    "id": "uuid-circle",
    "status": "open",
    "capacity": 3,
    "language": "fa",
    "pathId": "uuid-path",
    "exerciseId": "uuid-exercise",
    "ageBand": "teen",
    "membersCount": 1,
    "isOwner": true,
    "isActiveMember": true
  },
  "warning": false,
  "warningCode": null
}
```

- اگر کاربر دارای اشتراک فعال و این حلقه سومین حلقۀ او باشد:
  - `warning: true` و `warningCode: "MULTIPLE_ACTIVE_CIRCLES"` برگردانده می‌شود تا UI هشدار بنری نشان دهد.

**خطاها:**

- `400 Bad Request`:
  - `submissionId` خالی یا نامعتبر،
  - تمرینی که Critique برای آن فعال نیست (`CRITIQUE_NOT_ENABLED`).
- `403 Forbidden`:
  - Submission متعلق به user فعلی نیست (`FORBIDDEN`)،
  - کاربر زیر ۱۳ سال (در صورت عبور از لایۀ Identity).
- `404 Not Found`:
  - Submission یافت نشد (`SUBMISSION_NOT_FOUND`).
- `409 Conflict`:
  - محدودیت حلقه (`CIRCLE_LIMIT_REACHED`, `CIRCLE_LIMIT_REACHED_FOR_PATH`)،
  - حلقه در وضعیت نامناسب برای Join است.

---

### ۴.۲. مشاهدهٔ حلقه و وضعیت نقدها

**URL:**  
`GET /api/circles/my`

**Auth:**  
- نیازمند کاربر Authenticated.

**منطق:**

1. یافتن حلقه‌هایی که user در آن‌ها عضو است (`circle_members.user_id = currentUserId` و `left_at IS NULL`).
2. فیلترکردن حلقه‌هایی که:
   - `critique_circles.status IN ('open','active')`.
3. اگر چند حلقه یافت شد:
   - اولویت با حلقه‌ای که برای user هنوز `peer_feedback_unlocked=false` دارد؛
   - سپس جدیدترین `critique_circles.created_at`.
4. اگر هیچ حلقه‌ای یافت نشد:

```json
{
  "activeCircle": null
}
```

5. اگر حلقه یافت شد:
   - اعضا (از `circle_members` + `users.display_name`).
   - Submissions (از `circle_submissions` + `submissions`):
     - برای هر Submission:
       - اگر `authorId != currentUserId`:
         - `myFeedbackStatus` = `pending` یا `submitted` (بر اساس وجود `peer_feedback`).
       - اگر `authorId == currentUserId`:
         - `feedbackUnlocked = submissions.peer_feedback_unlocked`.
   - پاسخ:

```json
{
  "activeCircle": {
    "id": "uuid-circle",
    "status": "active",
    "capacity": 3,
    "language": "fa",
    "pathId": "uuid-path",
    "exerciseId": "uuid-exercise",
    "ageBand": "teen",
    "members": [
      { "userId": "u1", "displayName": "کاربر الف" },
      { "userId": "u2", "displayName": "کاربر ب" },
      { "userId": "me", "displayName": "من" }
    ],
    "submissions": [
      {
        "submissionId": "s1",
        "authorId": "u1",
        "authorDisplayName": "کاربر الف",
        "excerpt": "چند خط اول متن...",
        "myFeedbackStatus": "pending"
      },
      {
        "submissionId": "s2",
        "authorId": "u2",
        "authorDisplayName": "کاربر ب",
        "excerpt": "چند خط اول...",
        "myFeedbackStatus": "submitted"
      },
      {
        "submissionId": "s3",
        "authorId": "me",
        "authorDisplayName": "من",
        "excerpt": "چند خط اول متن من...",
        "feedbackUnlocked": false
      }
    ]
  }
}
```

---

### ۴.۳. ثبت نقد همتا (Submit Peer Feedback)

**URL:**  
`POST /api/peer-feedback`

**Auth:**  
- نیازمند کاربر Authenticated.

**Body (JSON):**

```json
{
  "circleId": "uuid",
  "submissionId": "uuid",
  "body": "متن نقد من...",
  "score": 4
}
```

**قواعد:**

1. `circleId`, `submissionId`, `body` الزامی‌اند.
2. کاربر باید عضو `circleId` باشد.
3. `submissionId` باید در `circle_submissions` برای همین `circleId` ثبت شده باشد.
4. `authorId` مربوط به آن Submission نباید `currentUserId` باشد (self-review ممنوع).
5. اگر رکوردی در `peer_feedback` با `(circleId, submissionId, reviewerId=currentUserId)` وجود داشته باشد:
   - خطای `409 Conflict` با کد `DUPLICATE_FEEDBACK`.
6. طول `body` باید بین ۲۰۰ و ۵۰۰۰ کاراکتر باشد.

**منطق:**

- درج رکورد جدید در `peer_feedback`.
- شمارش تعداد `peer_feedback`هایی که:
  - `circle_id = circleId`,
  - `reviewer_id = currentUserId`,
  - و `submission_id`شان متعلق به دیگران است (`authorId != currentUserId`).
- محاسبۀ N مؤثر برای این کاربر در این حلقه:

  ```text
  N_effective = min(N_base, count_of_other_submissions_for_user)
  ```

  - در MVP، `N_base = 2` (از Config مانند `CRITIQUE_REQUIRED_REVIEWS` خوانده می‌شود).
  - `count_of_other_submissions_for_user` = تعداد Submissionهایی در `circle_submissions` که `author_id != currentUserId` دارند.

- اگر `count_of_reviews_written >= N_effective`:
  - برای Submissionهای user در همان حلقه (`author_id = currentUserId`):
    - `peer_feedback_unlocked = TRUE` تنظیم می‌شود؛
  - `unlockedOwnFeedback = true` در پاسخ برگردانده می‌شود.

**پاسخ موفق (۲۰۱ Created):**

```json
{
  "success": true,
  "unlockedOwnFeedback": true
}
```

- اگر هنوز به حد نصاب نرسیده باشد: `unlockedOwnFeedback = false`.

**خطاها:**

- `400 Bad Request`:
  - ورودی ناقص،
  - متن نقد کمتر از ۲۰۰ یا بیشتر از ۵۰۰۰ کاراکتر (`FEEDBACK_TOO_SHORT` یا `FEEDBACK_TOO_LONG`).
- `403 Forbidden`:
  - user عضو حلقه نیست (`NOT_MEMBER`)،
  - self-review (`SELF_REVIEW_NOT_ALLOWED`).
- `404 Not Found`:
  - حلقه یا Submission در آن حلقه وجود ندارد.
- `409 Conflict`:
  - نقد تکراری (`DUPLICATE_FEEDBACK`).

---

### ۴.۴. گزارش تخلف در حلقه‌ها

**URL:**  
`POST /api/circles/report`

**Auth:**  
- نیازمند کاربر Authenticated.

**Body (JSON):**

```json
{
  "circleId": "uuid",
  "submissionId": "uuid",
  "peerFeedbackId": "uuid-optional",
  "reason": "دلیل کوتاه یا نوع تخلف (مثلاً توهین، محتوای نامناسب، اسپم)",
  "details": "توضیحات تکمیلی کاربر دربارهٔ مشکل"
}
```

**منطق:**

- Backend:
  - گزارش را با context `critique_circle_peer_feedback` در جدول `reports` ذخیره می‌کند.
  - متادادهٔ لازم (مانند `circleId`, `submissionId`, `peerFeedbackId`) را نگه می‌دارد.
- ناظر حقوقی:
  - از پنل Admin (`/admin/reports`) این گزارش‌ها را می‌بیند،
  - به داده‌های `peer_feedback` و `circle_submissions` مرتبط دسترسی دارد،
  - و بر اساس سیاست‌های [فصل پانزدهم دامنه](./product-domain-spec.md#chapter-15) اقدام می‌کند.

---

### ۴.۵. (اختیاری/فاز بعد) لیست همهٔ حلقه‌های کاربر

**این Endpoint برای MVP الزامی نیست، اما برای فازهای بعدی (به‌ویژه کاربران دارای اشتراک فعال) رزرو می‌شود.**

**URL پیشنهادی:**  
`GET /api/circles/all`

**خروجی نمونه:**

```json
{
  "circles": [
    {
      "circleId": "c1",
      "status": "active",
      "language": "fa",
      "pathId": "p1",
      "exerciseId": "e1",
      "ageBand": "adult",
      "membersCount": 3,
      "isActiveForUser": true
    },
    {
      "circleId": "c2",
      "status": "open",
      "language": "fa",
      "pathId": "p2",
      "exerciseId": "e5",
      "ageBand": "teen",
      "membersCount": 2,
      "isActiveForUser": false
    }
  ]
}
```

---

## ۵. فلو UX (User Experience Flow)

### ۵.۱. سناریوی اصلی کاربر

۱. کاربر تمرین را ارسال می‌کند (`submit`).
2. در صفحهٔ تأیید:
   - پیام «تمرین شما ارسال شد.»
   - اگر Critique برای آن تمرین فعال است: گزینه «پیوستن به حلقۀ نقد (Critique Circle)».
3. کلیک روی دکمه:
   - فرانت‌اند → `POST /api/circles/join` با `submissionId`.
   - Backend حلقه را پیدا/ایجاد می‌کند و اطلاعات آن را برمی‌گرداند.
   - UI پیام می‌دهد: «شما به یک حلقۀ نقد پیوستید…».
   - اگر کاربر teen باشد، حلقهٔ teen ساخته/انتخاب می‌شود؛ اگر adult باشد، حلقهٔ adult.
4. در `/[lang]/circles`:
   - اگر `status='open'`: پیام «در انتظار تکمیل اعضای حلقه هستیم.»
   - اگر `status='active'`: نمایش کارت تمرین‌های دیگران (با دکمهٔ «نوشتن نقد») و تمرین خود کاربر (با قفل، تا زمان نوشتن N نقد لازم براساس قاعدۀ پویا).
5. کاربر روی تمرین دیگران کلیک می‌کند:
   - UI متن تمرین + فرم نقد را نشان می‌دهد.
   - پس از ثبت نقد، کارت آن تمرین به حالت «نقد شده» می‌رود.
6. پس از نوشتن `N_effective` نقد معتبر (که می‌تواند در حلقۀ دو‌نفره ۱ و در حلقۀ سه‌نفره ۲ باشد):
   - Backend `peer_feedback_unlocked` را برای Submissionهای user در آن حلقه `true` می‌کند،
   - UI کارت تمرین user را از قفل خارج می‌کند و نقدهای هم‌حلقه‌ای‌ها را نشان می‌دهد.
7. در صورت مشاهدهٔ نقد نامناسب:
   - user روی دکمۀ «گزارش تخلف» کلیک می‌کند،
   - جزئیات را وارد می‌کند،
   - Backend `POST /api/circles/report` را صدا می‌زند.

---

## ۶. امنیت، حریم خصوصی و حذف حساب

### ۶.۱. Auth & RBAC

- تمام Endpointهای Critique Circles نیازمند Auth هستند (Login).
- استفاده از RBAC:
  - فقط نقش‌های `user` و بالاتر می‌توانند از این APIها استفاده کنند.
  - Admin و نقش‌های `legal` از طریق پنل Admin و با ثبت در AuditLog به داده‌ها دسترسی دارند.

### ۶.۲. محرمانگی محتوا

- متن تمرین‌ها و نقدها:
  - فقط برای اعضای همان حلقه و Adminهای مجاز قابل مشاهده است.
  - به‌صورت عمومی (مثلاً در صفحات ایندکس‌شونده) نمایش داده نمی‌شوند، مگر در قالب «نمونه‌های برگزیده» با رضایت صریح ([فصل نهم دامنه](./product-domain-spec.md#chapter-9)).

### ۶.۳. گزارش تخلف

- UI کاربر نیازی به دانستن شناسه‌های داخلی ندارد؛ این شناسه‌ها در پس‌زمینه ارسال می‌شوند (با context مناسب).
- سیستم گزارش‌ها را با context `critique_circle_peer_feedback` ثبت می‌کند تا تیم ناظر بتواند به‌طور هدفمندتر بررسی کند.

### ۶.۴. حذف حساب و تأثیر آن روی Critique Circles

مطابق [فصل هفدهم دامنه](./product-domain-spec.md#chapter-17):

۱. **حالت پیش‌فرض (ناشناس‌سازی مشارکت‌ها):**
   - `users` حذف می‌شود؛
   - در Critique Circles:
     - فیلدهای `circle_members.user_id`, `circle_submissions.author_id`, `peer_feedback.reviewer_id` به `NULL` ست می‌شوند؛
     - متن تمرین‌ها و نقدها در جداول باقی می‌ماند، اما هیچ پیوند فنی به user خاص ندارد؛
     - در UI، چنین مشارکت‌هایی با برچسب عمومی (مثلاً «یک کاربر») نمایش داده می‌شوند.
   - این حالت باعث حفظ دانش جمعی و مثال‌های مفید، بدون حفظ هویت فردی، می‌شود.

2. **حالت حذف کامل مشارکت‌ها:**
   - علاوه بر حذف حساب:
     - Submissionهای کاربر حذف می‌شوند (`ON DELETE CASCADE` در `circle_submissions` و `peer_feedback` برای `submission_id` عمل می‌کند)،
     - نقدهای دیگران روی تمرین‌های او نیز به‌دلیل وابستگی فنی ممکن است حذف شوند.
   - این رفتار در UI (هنگام انتخاب گزینهٔ «حذف کامل») به طور صریح توضیح داده می‌شود.

---

## ۷. سناریوهای لبه‌ای (Edge Cases) و نکات پیاده‌سازی

۱. **حلقه‌ای که هرگز به ظرفیت کامل نمی‌رسد:**
   - اگر حلقه‌ای مدت طولانی `open` بماند (مثلاً فقط ۲ نفر در آن باشند و ظرفیت ۳ باشد):
     - در MVP منطق خودکار برای ادغام یا بستن وجود ندارد؛
     - UI باید وضعیت را شفاف به کاربر بگوید («این حلقه هنوز کامل نشده، ممکن است کمی زمان ببرد تا هم‌حلقه‌ای‌ها پیدا شوند.»).
   - با قاعدۀ N پویا، اگر فقط دو نفر عضو باشند، `count_of_other_submissions_for_user = 1` و بنابراین `N_effective = 1` است؛  
     یعنی هرکدام از این دو نفر پس از نوشتن یک نقد معتبر روی تمرین نفر دیگر می‌تواند نقدهای دریافتی خود را ببیند، حتی اگر حلقه از نظر `status` همچنان `open` باقی مانده باشد.

۲. **ترک حلقه توسط کاربر (Leave Circle):**
   - در MVP فیچر Leave رسمی (برای خروج داوطلبانهٔ کاربر از حلقه) ندارد؛
   - خروج از حلقه در سطح داده تنها در دو حالت رخ می‌دهد:
     - از طریق حذف حساب کاربری (مطابق بند ۶.۴)،
     - یا از طریق مداخلهٔ Admin (حذف/ویرایش رکوردهای Critique Circles در پنل مدیریتی).
   - اگر در آینده فیچر Leave اضافه شود، باید اثر آن بر نقدهای داده‌شده/دریافت‌شده و محدودیت‌های حلقه به‌وضوح تعریف شود.

۳. **تغییر ظرفیت حلقه پس از ایجاد:**
   - در MVP مجاز نیست؛ تغییر capacity فقط روی حلقه‌های جدید اثر دارد.

۴. **تغییر `is_critique_enabled` برای تمرین فعال:**
   - اگر Admin Critique را روی تمرینی که حلقه‌های فعال دارد خاموش کند:
     - ایجاد حلقهٔ جدید برای آن تمرین متوقف می‌شود؛
     - حلقه‌های موجود می‌توانند طبق روال ادامه دهند.

۵. **حذف تمرین:**
   - در حالت حذف کامل مشارکت‌ها یا در سناریوهای Admin:
     - حذف Submission باعث حذف رکوردهای مرتبط در `circle_submissions` و `peer_feedback` می‌شود؛
     - این عملیات باید با احتیاط و با تأکید بر تأثیر آن روی حلقه‌ها انجام شود.

۶. **محدودیت‌های سنی و Join:**
   - اگر کاربر teen (۱۳–۱۷) برای Submission خود درخواست Join بدهد:
     - حلقهٔ جدید با `age_band='teen'` ساخته می‌شود یا حلقهٔ teen موجود انتخاب می‌گردد؛
   - اگر کاربر adult (۱۸+) Join بدهد:
     - حلقهٔ adult ساخته/انتخاب می‌شود؛
   - هرگونه تلاش برای Join کاربر teen به حلقۀ adult یا بالعکس در منطق Matching اساساً رخ نمی‌دهد، زیرا حلقه‌های موجود همگی `age_band` مشخص دارند و Create/Join فقط بر اساس age_band خود کاربر صورت می‌گیرد.

۷. **تغییر بازۀ سنی در میانهٔ راه (Teen → Adult):**
   - اگر کاربر در حین استفاده از سیستم به ۱۸ سالگی برسد:
     - حلقه‌های قبلی که در آن‌ها به عنوان Teen عضو بوده، بدون تغییر باقی می‌مانند؛
     - از زمان تغییر سن به بعد، هر حلقۀ جدیدی که Join می‌کند با `age_band='adult'` ایجاد/انتخاب می‌شود؛
     - این رفتار با سیاست سنی [فصل چهاردهم دامنه](./product-domain-spec.md#chapter-14) هم‌خوان است.

---

## ۸. قابلیت‌های رزرو‌شده برای آینده (owner و رویدادها)

### ۸.۱. `owner_id` و نقش `owner`

- `owner_id` در `critique_circles` و نقش `owner` در `circle_members` برای استفاده‌های آتی رزرو شده‌اند:
  - بستن دستی حلقه توسط owner،
  - مدیریت اعضا (حذف عضو مشکل‌دار)،
  - تعیین deadline برای حلقه.
- **در MVP:**
  - هیچ API/UI برای مدیریت حلقه توسط owner پیاده نمی‌شود؛
  - owner صرفاً سازندهٔ حلقه است و دسترسی ویژه‌ای ندارد؛
  - تمام مدیریت حلقه‌ها در صورت نیاز از طریق Admin Panel انجام می‌شود.

### ۸.۲. رویدادهای (Events/Webhooks) پیشنهادی برای فاز بعد

در فازهای بعد می‌توان رویدادهای زیر را به‌منظور Integrations یا Automation اضافه کرد:

- `circle.became_active` – زمانی که حلقه از `open` به `active` می‌رسد.
- `circle.all_reviews_done` – زمانی که همهٔ اعضا N_effective نقد را تکمیل کرده‌اند.
- `feedback.unlocked` – زمانی که نقدهای دریافتی کاربر unlock می‌شوند (`peer_feedback_unlocked=true`).

این رویدادها در MVP پیاده‌سازی نمی‌شوند، اما طراحی فعلی دیتابیس و API امکان افزودن آن‌ها را می‌دهد.

---

## ۹. Error Codes و قراردادهای خطا

برای یکپارچگی رفتار خطاها در Backend و Frontend، استفاده از Error Codeهای زیر توصیه می‌شود:

| کد | نام | توضیح |
|:---|:----|:------|
| `CIRCLE_LIMIT_REACHED` | محدودیت حلقه | کاربر عادی به حداکثر تعداد حلقه‌های فعال مجاز رسیده |
| `CIRCLE_LIMIT_REACHED_FOR_PATH` | محدودیت Path | کاربر دارای اشتراک در همان Path یک حلقۀ فعال دارد |
| `SUBMISSION_NOT_FOUND` | یافت نشد | Submission با شناسهٔ داده‌شده وجود ندارد |
| `SUBMISSION_NOT_SUBMITTED` | وضعیت نامعتبر | Submission هنوز در وضعیت `draft` است |
| `CRITIQUE_NOT_ENABLED` | غیرفعال | Critique برای این تمرین فعال نیست |
| `NOT_MEMBER` | عضویت | کاربر عضو این حلقه نیست |
| `SELF_REVIEW_NOT_ALLOWED` | خودنقدی | نقد روی تمرین خود مجاز نیست |
| `FEEDBACK_TOO_SHORT` | متن کوتاه | نقد کمتر از ۲۰۰ کاراکتر است |
| `FEEDBACK_TOO_LONG` | متن بلند | نقد بیش از ۵۰۰۰ کاراکتر است |
| `DUPLICATE_FEEDBACK` | تکراری | کاربر قبلاً روی این Submission در این حلقه نقد نوشته است |
| `MULTIPLE_ACTIVE_CIRCLES` | چند حلقه | کاربر با اشتراک فعال در ≥۳ حلقهٔ هم‌زمان فعال است (هشدار) |

این کدها باید همراه با پیام انسانی معنادار در JSON پاسخ خطا قرار گیرند تا Frontend بتواند پیام مناسب نمایش دهد.

---

این Spec، مرجع نهایی برای فیچر Critique Circles است و باید در کنار:

- `product-domain-spec.md` (فصل هشتم، چهاردهم و هفدهم)،
- `architecture-fortified-monolith.md` (بخش Community & Compliance)،
- `technical-brief.md` (بخش ۴.۴ و ۷)،
- `build-plan-stage5.md` (فاز ۲، ۴ و ۵)،

استفاده شود.  
هرگونه پیاده‌سازی باید دقیقاً با این قواعد هم‌خوان باشد و هر تغییر آتی نیازمند ADR و به‌روزرسانی هم‌زمان این سند و اسناد مرجع است.
```

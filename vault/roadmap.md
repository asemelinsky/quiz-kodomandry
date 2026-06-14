# Roadmap & Decisions Log — Quiz Kodomandry

> Живий документ. Тут фіксуються поточні ініціативи, зафіксовані рішення, виконані віхи. Доповнюється по ходу проєкту.

**Last update:** 2026-06-14 (Initiative #2 — Auth & Tracking)

---

## 🎯 Поточна ініціатива

### Initiative #2 — Auth + Tracking + Parent Reports
**Статус:** ✅ Plan locked · 🚧 Implementation pending
**Зафіксовано:** 2026-06-14
**ETA:** Phase 0-3 → 1.5-2 дні; повний rollout → ~3-4 дні

#### Цілі
1. Ідентифікувати учнів які проходять квізи
2. Зберігати кожне завершення з per-question логом (бали, час, помилки, мова, retry-каунтер)
3. Аналіз з часом: прогрес-графіки, heatmap слабких місць, streak
4. Автоматичні звіти батькам у Telegram після кожного квізу
5. Дашборд: учнівський (per-token) + адмінський (overview всіх)

#### Зафіксовані рішення

| # | Рішення | Альтернативи |
|---|---|---|
| 1 | **Архітектурний tier:** Tier 2 — Postgres + identification + dashboard | Tier 1 (localStorage only) / Tier 3 (+ magic link) |
| 2 | **База даних:** Supabase Postgres | Vercel Postgres / SQLite on VPS |
| 3 | **Ідентифікація:** один URL з 8-char token на учня (`?u=mtv-x7k2qp`) | пікер імен / PIN |
| 4 | **Метрики:** per-question log (block, task_idx, was_first_try, num_wrong_clicks, time_sec) | тільки score+time |
| 5 | **Стартовий сід:** усі 5 учнів — Матвій, Тімур, Франц, Дені, Злата | тільки Матвій / Матвій+Франц |
| 6 | **Rollout-стратегія:** L4 тестується першим → потім bulk-rollout у всі квізи | усі відразу / тільки нові |
| 7 | **Доставка URLів батькам:** ви отримуєте 5 URLів від мене → передаєте у чаті | /enroll сторінка / QR-коди |
| 8 | **Парент-репорти:** Telegram bot, повідомлення після кожного завершення | без бота, тільки веб |
| 9 | **Bot enrollment:** deep-link з webhook (`https://t.me/<bot>?start=<code>` → автоматичний linking) | ручне додавання chat_id / без бота |

#### Архітектура

```
┌─────────────┐       ┌──────────────────────┐      ┌──────────────┐
│  Quiz HTML  │──────▶│  Vercel Functions    │─────▶│  Supabase    │
│  +tracker.js│  POST │  /api/attempt        │      │  Postgres    │
└─────────────┘       │  /api/me             │      └──────────────┘
                      │  /api/dashboard      │              ▲
                      │  /api/admin/*        │              │
                      │  /api/tg-webhook     │              │
                      └──────────┬───────────┘              │
                                 │                          │
                                 ▼                          │
                      ┌──────────────────────┐             │
                      │  Telegram Bot API    │             │
                      │  sendMessage         │             │
                      │  (outgoing only)     │             │
                      └──────────────────────┘             │
                                                            │
┌─────────────────┐                                        │
│ /dashboard?u=t  │────────────────────────────────────────┘
│ /admin?key=...  │
└─────────────────┘
```

#### Schema (Supabase Postgres)

```sql
create table students (
  id uuid primary key default gen_random_uuid(),
  slug text unique not null,                -- 'matviy', 'frants'
  name text not null,                       -- 'Матвій'
  token text unique not null,               -- 'mtv-x7k2qp'
  default_lang text default 'en',
  parent_chat_id bigint,                    -- Telegram chat_id (NULL until enrolled)
  parent_enroll_code text unique,           -- 'mtv-enroll-x7k2qp'
  parent_email text,                        -- зарезервовано на майбутнє
  created_at timestamptz default now()
);

create table quiz_attempts (
  id uuid primary key default gen_random_uuid(),
  student_id uuid references students(id) on delete cascade,
  quiz_slug text not null,
  lang_used text not null,
  started_at timestamptz not null,
  finished_at timestamptz not null,
  first_try_correct int not null,
  total_q int not null,
  score_pct int not null,
  grade_12 int not null,
  time_sec int not null,
  user_agent text,
  created_at timestamptz default now()
);
create index on quiz_attempts (student_id, finished_at desc);

create table question_log (
  id bigserial primary key,
  attempt_id uuid references quiz_attempts(id) on delete cascade,
  q_index int not null,
  block text not null,
  task_idx int not null,
  variation_idx int not null,
  was_first_try boolean not null,
  num_wrong_clicks int not null,
  time_sec int not null
);
create index on question_log (attempt_id);

create view student_weak_topics as
select
  qa.student_id,
  qa.quiz_slug,
  ql.block,
  ql.task_idx,
  count(*) as attempts,
  round(100.0 * sum(case when ql.was_first_try then 1 else 0 end) / count(*)) as first_try_pct,
  round(avg(ql.num_wrong_clicks)::numeric, 2) as avg_wrong_clicks
from question_log ql
join quiz_attempts qa on ql.attempt_id = qa.id
group by qa.student_id, qa.quiz_slug, ql.block, ql.task_idx;
```

#### Структура коду

```
quiz/
├── api/                       ← NEW. Vercel serverless functions
│   ├── _supabase.js
│   ├── attempt.js             ← POST: save attempt + per-question
│   ├── dashboard.js           ← GET: dashboard data per student
│   ├── me.js                  ← GET: student by token
│   ├── tg-webhook.js          ← POST: handle /start <code>
│   └── admin/
│       ├── students.js
│       └── attempts.js
├── dashboard/                 ← NEW. Static UI
│   ├── index.html             ← /dashboard?u=<token>
│   └── admin.html             ← /admin?key=<env>
├── public/
│   └── tracker.js             ← NEW. Inject у всі квізи
├── quizzes/                   ← existing
├── categories/                ← existing
└── vault/                     ← existing
```

#### API endpoints

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/me?token=...` | GET | token | name + default_lang |
| `/api/attempt` | POST | token in body | save attempt + question_log; trigger TG report |
| `/api/dashboard?token=...` | GET | token | history + aggregates + weak topics |
| `/api/tg-webhook` | POST | Telegram secret | handle bot updates (`/start <code>`) |
| `/api/admin/students?key=...` | GET | env key | all students summary |
| `/api/admin/attempts?key=...&student=...` | GET | env key | all attempts of a student |

#### Telegram report format

```
🎉 Матвій завершив квіз!

📚 L4 — Текстові задачі на частку
🕒 21 хв 34 сек
✅ Бал: 11/12 (92%) — 27/30 з 1-ї спроби
🌐 Мова: English
🔥 Streak: 4 дні поспіль

💡 Слабкі місця:
• Block C (знайти ціле) — Q1: 2 неправильні кліки
• Block B (залишок) — Q5: з підказкою
• Block A (¾ від цілого) — Q3: 1 помилка

📈 Прогрес: 9/12 → 11/12 ⬆️

🎯 Наступний: L5 Відсотки (план: 17.06)
🔗 Деталі: https://quiz-kodomandry.vercel.app/dashboard?u=<token>
```

#### Rollout phases

| # | Phase | Estimated | Dependencies |
|---|---|---|---|
| 0 | Supabase setup, schema deploy, env vars у Vercel | 1 год | — |
| 1 | API: `/me`, `/attempt`, `/dashboard` | 4 год | Phase 0 |
| 2 | `tracker.js` + інжект у L4 для тесту | 3 год | Phase 1 |
| 3 | Dashboard UI (учнівський) | 4 год | Phase 1 |
| 4 | Bulk інжект у решту квізів | 2 год | Phase 2 успішна |
| 5 | Адмін view + експорт CSV | 3 год | Phase 1 |
| 6 | Polish: heatmap, recommendations, streak | 4 год | Phase 3 |
| **7** | **Telegram bot + enrollment webhook + auto-report** | **3-4 год** | Phase 1 |

**Реалістичний MVP:** Phase 0-3 + 7 → перші реальні квізи трекаються і батьки отримують звіти за 2-3 дні.

#### Безпека & privacy

- Token у URL — security through obscurity. Прийнятно для дитячих даних; батьки контролюють розповсюдження.
- Supabase service-role key — тільки на бекенді (Vercel env vars), фронтенд не має доступу.
- Row Level Security на Supabase: усе закрите для anon role.
- Telegram bot token у Vercel env var `TG_BOT_TOKEN`.
- Admin key у `ADMIN_KEY` env var.
- GDPR/COPPA: дані діток зберігаються мінімально (ім'я, прогрес, час); ніяких PII (email, телефон); parent_chat_id — bigint від Telegram.

#### Open questions (не блокують MVP)

- Локалізація десяткового сепаратора у задачах (UK/ES кома vs EN крапка) — TODO у `methodology.md`.
- Rate limiting на API — не критично для MVP, додати потім.
- Backup стратегія Supabase даних — Supabase робить автоматично на free tier.
- Streak логіка: рахуємо календарні дні чи проходження поспіль? — вирішимо при дашборді.
- Recommendations engine: статичний список «наступний за планом» vs ML — поки що статичний.

---

## 📋 Backlog (ідеї, не зафіксовані рішення)

- **Drill-fiesta authoring:** автоматично генерувати drill-квізи з найслабшого topic learned analytics
- **Sibling-mode:** одна сім'я з кількома дітьми (Матвій + майбутні брати/сестри) — групувати у dashboard
- **Achievement badges:** geymification — "10 квізів у місяць", "Perfect score 3 разів поспіль"
- **Push notifications** замість Telegram — Web Push для прямих сповіщень у браузер
- **Експорт PDF** результатів для друку / батьківських зборів
- **A/B-тестинг hint quality:** який hint format дає кращий learning outcome
- **Teacher mode:** дашборд для вчителя з усім класом (не тільки сім'я)
- **Voice mode:** TTS-озвучка задач для слабкобачачих

---

## ✅ Виконані initiatives

### Initiative #1 — Trilingual baseline (UK/EN/ES) — 2026-06-14
- L3 (Пропорції) + L4 (Частка) переписані з трилінгвом UK/EN/ES
- `<select>` dropdown UI (відновлений канон L1)
- EN за замовчуванням для Матвія L3+
- Methodology оновлена: `vault/methodology.md` секція «🌐 МОВНІ ПРАВИЛА»
- Reference quiz: L1 `drobi-arifmetyka-7klas-ispanska-2026-05-31`

### Прозорий status quo (станом на 2026-06-14)

**Live квізів:** 17 (Матвій 6 + Тімур 10 Phase A + Франц 1 + Злата 1 + Дені 1 + exam-revision 2)

**Учні у системі (плани):**
- Матвій — 27-урочний літній план (Phase 1: L1-L9 шкільний фундамент, Phase 2: L10-L26 літо)
- Тімур — Phase A 10 квізів (en math vocabulary, готові)
- Франц — 10-урочний план (L2 готово, plan-frants.md)
- Злата — 5 клас, 1 sample готовий
- Дені — Year 8 British, 1 sample готовий

**Інфра:**
- Vercel auto-deploy з GitHub: `asemelinsky/quiz-kodomandry`
- Domain: `quiz-kodomandry.vercel.app`
- Стек: single-file HTML, без backend (зараз), CSP-friendly

---

## 📝 Decisions Log (хронологічно, новіші зверху)

| Date | Decision | Why |
|---|---|---|
| 2026-06-14 | Initiative #2 (Auth + Tracking) plan locked | потреба бачити прогрес дітей за часом |
| 2026-06-14 | Усі квізи Матвія MUST бути трилінгві UK/EN/ES | методологічна узгодженість, плутанина після MVP |
| 2026-06-14 | EN default для Матвія L3+ | британська школа, англомовне викладання |
| 2026-06-14 | `<select>` dropdown для language switcher (не buttons) | reference у L1, масштабується на 4+ мов |
| 2026-06-10 | Topic-tag на фінальному екрані всіх квізів | за побажанням тата Матвія, скриншоти не показували який тест |
| 2026-06-10 | Theory section вбудована у start screen | за побажанням тата Матвія, як шпаргалка перед drill |
| 2026-06-10 | Urgent exam-revision drill як override планового L4 | іспит наступного дня, важливіше за плановий урок |
| 2026-06-09 | Дені (8 клас British) додано як sample | запит користувача на головну сторінку |
| 2026-06-09 | Злата (5 клас) додано як sample | запит користувача |
| 2026-06-08 | Франц 10-урочний план + L2 sample | новий учень з Німеччини |
| 2026-06-07 | Default UI lang Матвія: EN (раніше було UK помилково) | уточнили що Матвій у британському коледжі |
| 2026-05-31 | Hybrid format: wrong → hint → retry, score тільки за 1-ю спробою | оптимальний для drill-to-automaticity |
| 2026-05-31 | 30-question стандарт (3 × 10 блоки) для Матвія | методологія, виключно для дрилу |
| 2026-05-17 | Старт проєкту з pre-plan квізами для Матвія | base для подальших ітерацій |

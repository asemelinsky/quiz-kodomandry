# Architecture — структура проєкту і навігація

> Як влаштований Quiz Hub зсередини: файли, маршрути, потік навігації, де що живе.

## TL;DR

```
                    ┌─────────────────────┐
                    │   /  (Quiz Hub)     │  ← 4 категорії
                    └──────────┬──────────┘
                               │
       ┌───────────────────────┼───────────────────────┐
       ▼                       ▼                       ▼
 /categories/matviy   /categories/timur     /categories/python
  Матвій (27 уроків)  Тімур (20 днів EN)    Python (Кодомандри)
       │                       │                       │
       ▼                       ▼                       ▼
  /quizzes/<slug>         /en-math/<slug>         /quizzes/<slug>
  (живі HTML квізи)       (живі HTML квізи)       (живі HTML квізи)
```

3 рівні: **Hub → категорія → квіз**. Усе статичне, без бекенду.

## Файлова структура

```
quiz/
├── index.html              ← Quiz Hub (головна, 4 категорії)
├── onboarding.html         ← 🎓 Артефакт для Тимура (диплом, не квіз)
├── vercel.json             ← cleanUrls + cache headers
├── assets/
│   └── hub.css             ← Shared CSS для index + categories
├── categories/             ← Підсторінки по учнях/програмах
│   ├── matviy.html         ← 🧮 Матвій
│   ├── timur.html          ← 📐 Тімур (English math vocab)
│   └── python.html         ← 🐍 Python для Кодомандрів
├── quizzes/                ← Самі квізи (single-file HTML)
│   ├── drobi-arifmetyka-7klas-ispanska-2026-05-31.html      (L1)
│   ├── msd-msm-podilnist-7klas-ispanska-2026-06-03.html     (L2)
│   ├── drobi-chas-7klas-ispanska-2026-05-20.html            (pre-plan)
│   ├── drobi-desyatkovi-vidsotky-7klas-ispanska-2026-05-27.html (pre-plan)
│   ├── zagalna-7klas-ispanska-2026-05-17.html               (pre-plan)
│   └── return-perehony-urok1-2026-05-17.html                (Python)
├── en-math/                ← Англомовні квізи для Тімура
│   └── d01-linear-algebra-2026-05-31.html                   (D1)
├── reports/                ← Артефакти (PNG, тощо)
│   └── 2026-05-31-matviy-plan.png  ← Інфографіка для батька
├── vault/                  ← "Робоча пам'ять" для Claude (не публічне)
│   ├── README.md           ← Index vault'а
│   ├── plan.md             ← 27-урочний план з статусами
│   ├── methodology.md      ← Стандарт квізу, слабкі місця Матвія
│   └── lessons/            ← Per-lesson нотатки
│       ├── L01-2026-05-31-drobi-arifmetyka.md
│       └── L02-2026-06-03-msd-msm-podilnist.md
├── docs/                   ← Документація проєкту (цей файл тут)
│   ├── README.md           ← (не існує, корінь у /README.md)
│   ├── architecture.md     ← цей файл
│   ├── workflow.md         ← Як додати новий квіз
│   └── quiz-map.md         ← Каталог-карта (опубл. на bajka.pp.ua)
├── CLAUDE.md               ← Методистська роль для Claude-сесії
└── README.md               ← Топ-рівневий огляд
```

## Маршрути (Vercel)

`vercel.json` має `"cleanUrls": true` — `.html` суфікс прибирається.

| URL | Файл |
|---|---|
| `/` | `index.html` |
| `/onboarding` | `onboarding.html` |
| `/categories/matviy` | `categories/matviy.html` |
| `/categories/timur` | `categories/timur.html` |
| `/categories/python` | `categories/python.html` |
| `/quizzes/<slug>` | `quizzes/<slug>.html` |
| `/en-math/<slug>` | `en-math/<slug>.html` |
| `/assets/hub.css` | `assets/hub.css` (без cleanUrls — це asset) |

Кеш-header `public, max-age=3600, must-revalidate` встановлюється для `/quizzes/(.*)` (1 година). Решта — Vercel defaults.

## Потік навігації

1. Користувач відкриває **/** → Quiz Hub з 4 картками-категоріями (📚 Hub).
2. Клікає на 🧮 Матвій → **/categories/matviy** — список 30 квізів з його програми, згрупованих за фазами (Pre-plan / Phase 1 / Phase 2 / Phase 3). Кожен L# — окрема картка зі статусом.
3. Клікає на живий L1 → **/quizzes/drobi-arifmetyka-7klas-ispanska-2026-05-31** — сам інтерактивний квіз.
4. Pending квізи (статус ⏳) показані сірим, не клікабельні (`pointer-events: none`).

## Стандарт квізу

Per `vault/methodology.md`:

- 30 завдань = 3 блоки × 10 (drill-to-automaticity)
- Hybrid format: неправильно → підказка → ретрай, бал тільки за з 1-ї спроби
- 12-бальна шкала, таймер
- UK/EN/ES з locale-aware десятковим розділювачем (кома для uk/es, крапка для en)
- 3 числові варіації на завдання + перемішування позицій відповідей
- Дистрактори — моделі типових помилок учня (для Матвія: нескорочений дріб, невиділена ціла, плутання НСД↔НСК)
- Single-file HTML, без зовнішніх залежностей, працює офлайн
- System font stack (cross-platform Cyrillic)

Шаблон-стартер: `quizzes/drobi-arifmetyka-7klas-ispanska-2026-05-31.html` (L1) — повний приклад data shape (SECTIONS / GEN / explain / i18n) для арифметичних задач з мішаними дробами.

## Кольорове кодування фаз (для UI квізів)

- **Phase 1 (шкільний фундамент):** indigo `#4f46e5 → #6366f1`
- **Phase 2 (літо):** teal `#0d9488 → #14b8a6`
- **Phase 3 (фінал):** amber `#d97706 → #f59e0b`
- **Drill-fiesta уроки:** виділені помаранчевим бейджем у навігації

Hub + categories pages — індиго (Phase 1 brand), щоб мати єдиний візуальний голос.

## Vault — "робоча пам'ять"

Vault (`/vault/`) — це **не публічна частина сайту**, хоча комічена у git. Вона потрібна для майбутніх Claude-сесій, щоб:

- Бачити статус кожного уроку (`plan.md`) без читання усіх HTML'ів
- Розуміти методологію — `methodology.md` для **Матвія**, `methodology-en-math.md` для **Тімура**
- Звірятись з `authorities-en-math.md` що які джерела вже використано для математичних означень (Тімур-вимога)
- Звірятись з `lessons/L<NN>-...md` що **які числа і дистрактори** вже були використані — щоб не повторити у Drill-fiesta (L12, L16, L20, L24)

Lesson notes мають секцію «Чого НЕ було» — це навмисний след для майбутнього drill-планування.

## Notes-hub публікація (bajka.pp.ua)

Каталог-карта `docs/quiz-map.md` дзеркалиться на `https://bajka.pp.ua/notes/infra/quiz-map/` (через спільний mkdocs-build з `/root/projects/docs/`). Detailed setup — у `/root/projects/docs/shared-services.md`.

Trigger rebuild:
```bash
cd /root/projects/docs-site/infra && /tmp/mkdocs-venv/bin/mkdocs build
```
(або чекай ~15 хв cron з host VPS)

## Деплоймент

- **GitHub:** https://github.com/asemelinsky/quiz-kodomandry (public)
- **Branch `main`** → production
- **Push у main → Vercel auto-deploy за 5-30 с**
- **Push у non-main branch** → preview URL `quiz-kodomandry-git-<branch>-...vercel.app`
- Без build step (pure static)
- Vercel project: `quiz-kodomandry` у team `oleksiys-projects-1e19468f`

## Naming conventions

**Quiz HTML файли** (in `/quizzes/`): `<тема>-<клас>[-<програма>]-<дата>.html`

- `<тема>` — латиницею (drobi, msd-msm-podilnist, proportsii, geometriya)
- `<клас>` — `<N>klas` (7klas)
- `<програма>` — `ispanska` для Матвія (1º ESO). Пропускається для звичайної української програми.
- `<дата>` — `YYYY-MM-DD`

**English math quizzes** (in `/en-math/`): `d<NN>-<domain>-<date>.html`

- `<NN>` — двозначний номер дня (d01-d20)
- `<domain>` — латиницею (linear-algebra, calculus, probability)
- `<date>` — `YYYY-MM-DD`

**Lesson notes** (in `/vault/lessons/`): `L<NN>-<date>-<slug>.md`

- `<NN>` — двозначний номер уроку для Матвія (L01-L27)
- Або `D<NN>` для Тімура

## Що далі

Як саме створювати новий квіз і оновлювати навігацію — у [workflow.md](workflow.md).

# Quiz Hub — інтерактивні квізи для учнів

> Static site з персоналізованими інтерактивними квізами. 3-рівнева навігація: **Hub → категорія учня → квіз**. Single-file HTML квізи (HTML+CSS+JS inline, працюють офлайн).

🚀 **Live:** https://quiz-kodomandry.vercel.app/

## Що тут є зараз

| Категорія | URL | Що це |
|---|---|---|
| 🧮 **Матвій · математика** | [/categories/matviy](https://quiz-kodomandry.vercel.app/categories/matviy) | 7 клас, іспанська школа (1º ESO) · 27-урочний літній план повторення + pre-plan квізи |
| 📐 **Тімур · English math vocab** | [/categories/timur](https://quiz-kodomandry.vercel.app/categories/timur) | B2/C1 prep для Master's interview (data theory, U. Ulm) · 20-денна програма |
| 🐍 **Python · Кодомандри** | [/categories/python](https://quiz-kodomandry.vercel.app/categories/python) | Уроки для учнів школи Кодомандри |
| 🎓 **Онбординг диплома** | [/onboarding](https://quiz-kodomandry.vercel.app/onboarding) | Артефакт для Тимура (walkthrough AI-помічника, не квіз) |

## Структура (high-level)

```
quiz/
├── index.html              ← Quiz Hub (головна сторінка з 4 категоріями)
├── assets/hub.css          ← Shared CSS для навігації
├── categories/             ← Підсторінки по учнях/програмах
├── quizzes/                ← Math quizzes (single-file HTML)
├── en-math/                ← English math vocabulary quizzes
├── reports/                ← Артефакти (PNG-інфографіка, тощо)
├── vault/                  ← "Робоча пам'ять" Claude (плани, методологія, lesson notes)
├── docs/                   ← Документація (architecture, workflow, quiz-map)
└── vercel.json             ← cleanUrls + cache headers
```

Детальна структура — [docs/architecture.md](docs/architecture.md).

## Стандарт квізу

- **30 завдань** = 3 блоки × 10 (drill-to-automaticity)
- **Hybrid format:** неправильно → підказка → ретрай, бал тільки за з 1-ї спроби
- **12-бальна** оцінка + таймер
- **UK/EN/ES** з locale-aware десятковим розділювачем
- **3 числові варіації** на завдання + перемішування позицій відповідей
- **Дистрактори** моделюють реальні помилки конкретного учня
- **Single-file HTML**, без зовнішніх залежностей

Філософія — [vault/methodology.md](vault/methodology.md).

## Документація

| Файл | Призначення |
|---|---|
| [docs/architecture.md](docs/architecture.md) | Файлова структура, маршрути, потік навігації, конвенції назв |
| [docs/workflow.md](docs/workflow.md) | Покроковий процес додавання нового квізу |
| [docs/quiz-map.md](docs/quiz-map.md) | Каталог-карта всіх квізів (також публікується на [bajka.pp.ua/notes/infra/quiz-map](https://bajka.pp.ua/notes/infra/quiz-map/)) |
| [vault/methodology.md](vault/methodology.md) | Стандарт квізу, слабкі місця учня, патерни дистракторів |
| [vault/plan.md](vault/plan.md) | 27-урочний план для Матвія з статусами |
| [vault/lessons/](vault/lessons/) | Per-lesson нотатки з зафіксованими задачами |
| [CLAUDE.md](CLAUDE.md) | Методистська роль для Claude-сесії |

## Як додати новий квіз

Швидко:
1. Подивись `vault/plan.md` — наступний L# у статусі ⏳
2. Згенеруй `quizzes/<slug>.html` за стандартом
3. Онови `categories/<кат>.html` (pending → live)
4. Онови `vault/plan.md` (статус ✅) + створи `vault/lessons/L##-...md`
5. `git add . && git commit && git push` → Vercel auto-deploy 5-30с

Повний процес — [docs/workflow.md](docs/workflow.md).

## Стек

- Pure static (без Next.js / build step)
- Vercel hosting · GitHub auto-deploy з `main` branch
- Public repo: https://github.com/asemelinsky/quiz-kodomandry

## Privacy

Public repo + public deploy. Жодних особистих даних, API tokens, IPs — лише HTML з навчальними матеріалами.

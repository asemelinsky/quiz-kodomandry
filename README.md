# Quiz — інтерактивні математичні вікторини

> Static site з HTML-вікторинами з математики для учнів 5-12 класів. Один файл = один quiz (HTML+CSS+JS inline, працюють офлайн).

🚀 **Live:** https://quiz-kodomandry.vercel.app/ (custom domain `quizzes.bajka.pp.ua` — у roadmap)

## Структура

```
quiz/
├── index.html        ← landing-сторінка з переліком вікторин
├── vercel.json       ← Vercel config (clean URLs, caching)
├── quizzes/          ← збережені готові .html вікторини
├── docs/
│   └── deploy-vercel.md   ← workflow публікації нового quiz'а
├── CLAUDE.md         ← методистська роль (для AI-генерації)
└── README.md
```

## Як додати новий quiz

Коротко — повний workflow у `docs/deploy-vercel.md`:

1. Створи `quizzes/<slug>.html` (через Claude-сесію з cwd=`/root/projects/quiz/` за роллю з `CLAUDE.md`)
2. Додай посилання у `index.html` (між `<!-- QUIZZES_START -->` і `<!-- QUIZZES_END -->`)
3. `git add . && git commit -m "feat: <slug> quiz" && git push`
4. Vercel auto-deploy за 30-60 секунд

## Стек

- Pure static (без Next.js / build step)
- Vercel hosting (project `quiz-kodomandry` у `oleksiys-projects-1e19468f`)
- GitHub: https://github.com/asemelinsky/quiz-kodomandry (public)
- Branch: `main` (production)

## Не sensitive

Public repo + public deploy. Жодних особистих даних, API tokens, IPs — це просто HTML з математичними задачами.

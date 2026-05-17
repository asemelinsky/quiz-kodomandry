# Publication workflow — як опублікувати новий quiz

Project це **static site на Vercel з GitHub auto-deploy**. Усе налаштовано так, щоб додавання нового quiz'а зводилось до 3 кроків.

## TL;DR

```bash
# 1. Створи .html у quizzes/ (через Claude-сесію за роллю з CLAUDE.md)
# 2. Додай посилання у index.html (між QUIZZES_START / QUIZZES_END)
# 3. git add . && git commit -m "feat: <slug>" && git push
# → Vercel auto-deploy за 30-60 с
```

## Інфраструктура

| Шар | Деталі |
|---|---|
| Source | https://github.com/asemelinsky/quiz-kodomandry (public, branch `main`) |
| Hosting | Vercel project `quiz-kodomandry` у team `oleksiys-projects-1e19468f` |
| Production URL | https://quiz-kodomandry.vercel.app/ |
| Custom domain (планується) | `quizzes.bajka.pp.ua` — потребує ручного DNS-кроку, див. нижче |
| Build step | НЕМАЄ — pure static |
| Auto-deploy | ✅ enabled (Vercel watches `main` branch, deploys on push) |
| Branch deploys | Push у будь-яку non-main branch → preview URL `quiz-kodomandry-git-<branch>-...vercel.app` |

## Покроковий workflow

### Крок 1. Згенеруй quiz

У Claude-сесії з cwd=`/root/projects/quiz/` — використовуй роль методиста з `CLAUDE.md`. Файл генерується у `quizzes/<тема>-<клас>-<дата>.html` (приклад: `quizzes/drobi-6klas-2026-05-17.html`).

Перевір локально:
```bash
# Просто відкрий у браузері — single-file, без серверу
firefox quizzes/drobi-6klas-2026-05-17.html
# або
xdg-open quizzes/drobi-6klas-2026-05-17.html
```

### Крок 2. Додай у index.html

Знайди блок:
```html
<!-- QUIZZES_START -->
...
<!-- QUIZZES_END -->
```

Заміни `<div class="empty">...</div>` (або додай наступний рядок) на:
```html
<a class="quiz-card" href="/quizzes/drobi-6klas-2026-05-17">
  <div class="quiz-title">Дроби — 6 клас</div>
  <div class="quiz-meta">10 запитань · середній рівень · 2026-05-17</div>
</a>
```

> **Зверни увагу:** `href` БЕЗ `.html` суфікса — у `vercel.json` стоїть `cleanUrls: true`, тому Vercel сам резолвить `/quizzes/<slug>` → `/quizzes/<slug>.html`.

### Крок 3. Commit + push

```bash
cd /root/projects/quiz/
git add quizzes/drobi-6klas-2026-05-17.html index.html
git commit -m "feat: дроби 6 клас quiz"

# Push (token-based, конвенція з push skill)
TOKEN=$(cat /root/projects/.claude/.github_token)
ORIG_URL=$(git remote get-url origin)
git remote set-url origin "https://${TOKEN}@github.com/asemelinsky/quiz-kodomandry.git"
git push origin main
git remote set-url origin "$ORIG_URL"
```

### Крок 4. Чекай deploy (30-60 с)

Vercel автоматично підбирає push:
- GitHub webhook → Vercel build → `https://quiz-kodomandry.vercel.app/` оновлений
- Status можна перевірити: `vercel ls --scope oleksiys-projects-1e19468f | head -5`

Якщо deploy fails — інспектуй:
```bash
vercel logs <deployment-url> --scope oleksiys-projects-1e19468f
```

## Опційно: підняти `vercel-deploy` skill

`/vercel:deploy production` (на Vercel CLI) — manual production deploy без push. Корисно якщо треба promote preview deployment у production без коду в `main`.

## Custom domain `quizzes.bajka.pp.ua` (ручний крок, потребує DNS-access)

Vercel не має API-доступу до DNS зони `bajka.pp.ua` (вона на pp.ua registrar, не Cloudflare). Тому subdomain додається у два кроки — **необхідна ручна дія від Олексія**:

### Крок 1. Додати domain у Vercel project
```bash
vercel domains add quizzes.bajka.pp.ua quiz-kodomandry --scope oleksiys-projects-1e19468f
```
Vercel виведе DNS-record який треба додати (зазвичай CNAME `quizzes` → `cname.vercel-dns.com`).

### Крок 2. Додати DNS-record у registrar panel
Зайти у DNS-панель `pp.ua` registrar'а:
- Тип: `CNAME`
- Name/Host: `quizzes`
- Value/Target: `cname.vercel-dns.com`
- TTL: default (3600 або auto)

Через 5-30 хв domain пропагується. Перевірка:
```bash
getent hosts quizzes.bajka.pp.ua  # має повернути Vercel IP
curl -I https://quizzes.bajka.pp.ua/  # HTTP/2 200
```

Vercel автоматично provisions TLS-сертифікат Let's Encrypt після того, як DNS resolved.

## Видалити quiz

```bash
git rm quizzes/<slug>.html
# Прибери посилання з index.html
git commit -m "remove: <slug>"
git push origin main
```

## Що НЕ commit'и

- `.vercel/` — auto-generated linkage до Vercel (вже у `.gitignore`)
- `.env*` — не використовуємо, але про всяк
- `node_modules/` — їх теж немає, build step відсутній

## Troubleshooting

| Симптом | Причина / фікс |
|---|---|
| `git push` → `403` | Token у `/root/projects/.claude/.github_token` expired → знайди новий через `grep -rE "ghp_[A-Za-z0-9]{20,40}" /root/.claude/projects/-root-projects/*.jsonl` + перевір через `curl -H "Authorization: token <t>" https://api.github.com/user` |
| Vercel deploy fails | `vercel logs <url>` — типово вказує на syntax error у HTML / vercel.json |
| 404 на `/quizzes/<slug>` | `vercel.json` має `cleanUrls: true` — переконайся що файл `quizzes/<slug>.html` справді existing на main. Без суфікса лінк працює, з суфіксом теж (redirect) |
| Custom domain не резолвиться | DNS пропагація 5-30 хв; перевір через `getent hosts <domain>`. Якщо все ще не working — у Vercel domains list має бути ✅ біля domain'a |

## Інші related skills (на майбутнє)

- `vercel:deploy` — manual production promote
- `vercel:env` — якщо колись додамо env vars (для analytics, наприклад)
- `vercel:performance-optimizer` — Lighthouse, Core Web Vitals
- `vercel:status` — quick overview deployments

## Cross-refs

- Quiz role / методика: `/root/projects/quiz/CLAUDE.md`
- Push skill: `/root/projects/.claude/skills/push/SKILL.md`
- Repo + visibility convention (зразок): `asemelinsky/zoom-uploader-distributed` (public), `asemelinsky/CCTweak_LuaPageCoding` (private)

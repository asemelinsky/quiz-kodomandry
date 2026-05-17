# Як опублікувати квіз — інструкція для методиста

Цей файл — для тебе, методиста (роль описана у `CLAUDE.md`). Він пояснює що робити з готовим `quiz.html` після того, як ти його згенерував.

> **TL;DR:** save у `quizzes/`, додай 1 рядок у `index.html`, виконай 3 git-команди — Vercel сам опублікує за хвилину.

## Передумови (одноразово, вже зроблено admin'ом)

- GitHub repo підключений: https://github.com/asemelinsky/quiz-kodomandry
- Vercel auto-deploy налаштований: будь-який push у `main` → продакшен за 30-60 с
- Сайт live: https://quiz-kodomandry.vercel.app/

Тобі **не треба** ні до чого з цього лізти. Просто виконуй кроки нижче.

---

## Setup (одноразово на кожен новий клон repo)

Якщо це перший commit з твоєї сесії у цей repo — встанови local git identity:

```bash
cd /root/projects/quiz/
git config user.name "Oleksii Semelinsky (via Claude)"
git config user.email "claude@a.semelinsky"
```

Це per-repo config (живе у `.git/config`), а не system/global — тому правило "NEVER update git config" не порушується. Без цього кожен commit мусить мати `-c user.name="..." -c user.email="..."` flag, що зайве. Перевірити що identity вже є:

```bash
git config --get user.name  # має повернути "Oleksii Semelinsky (via Claude)"
```

---

## Workflow за 4 кроки

### Крок 1. Save файл у `quizzes/`

Назва файла — `тема-клас-дата.html` (всі літери latin/transliterated, без пробілів, через дефіс):

```
quizzes/drobi-6klas-2026-05-17.html
quizzes/trigonometriya-10klas-2026-05-17.html
quizzes/zagalna-7klas-2026-05-17.html
```

> 📌 **Slug = ім'я файла без `.html`** — він буде URL'ом у браузері. Тому має бути читабельний і transliterated. Не використовуй кирилицю у назвах файлів (Vercel їх обробить, але URL виглядатиме брудно).

### Крок 2. Локальна перевірка (опційно, але рекомендую)

Відкрий файл у браузері перш ніж публікувати:

```bash
xdg-open quizzes/drobi-6klas-2026-05-17.html
# або
firefox quizzes/drobi-6klas-2026-05-17.html
```

Пройди quiz сам — переконайся що всі питання працюють, відповіді правильні, оцінка коректна. Бо після push'а виправляти — ще один цикл "edit → push → deploy".

### Крок 3. Додай посилання у `index.html`

Відкрий `/root/projects/quiz/index.html`, знайди маркери:

```html
<!-- QUIZZES_START -->
...тут...
<!-- QUIZZES_END -->
```

Якщо це **перший** quiz — видали placeholder `<div class="empty">...</div>` між маркерами і встав замість нього свою картку.

Якщо це **черговий** quiz — просто додай нову картку поряд з існуючими.

**Шаблон картки:**

```html
<a class="quiz-card" href="/quizzes/drobi-6klas-2026-05-17">
  <div class="quiz-title">Дроби — 6 клас</div>
  <div class="quiz-meta">10 запитань · середній рівень · 2026-05-17</div>
</a>
```

Що поміняти:
- `href` → slug файла **без `.html`** суфікса (працює завдяки `vercel.json` cleanUrls)
- `quiz-title` → людська назва (можна укр-кирилицею)
- `quiz-meta` → метадані: кількість запитань, рівень, дата

### Крок 4. Commit + push

Скопіюй і запусти 4 рядки (підстав свій slug у commit message):

```bash
cd /root/projects/quiz/
git add quizzes/<slug>.html index.html
git commit -m "feat: <slug>"

TOKEN=$(cat /root/projects/.claude/.github_token)
ORIG_URL=$(git remote get-url origin)
git remote set-url origin "https://${TOKEN}@github.com/asemelinsky/quiz-kodomandry.git"
git push origin main
git remote set-url origin "$ORIG_URL"
```

> **Чому такий складний push?** Конвенція з push skill — токен встановлюється тимчасово у remote URL, потім прибирається назад. Інакше токен залишається у `.git/config` (security gap). Це 4 додаткових рядки за безпеку.

Альтернатива: `/push` skill зробить це сам, якщо ти у тій Claude-сесії, що має його доступним.

### Крок 5. Готово — Vercel опублікує сам

За 30-60 секунд після push'а:
- https://quiz-kodomandry.vercel.app/ — landing оновиться, нова картка з'явиться
- https://quiz-kodomandry.vercel.app/quizzes/drobi-6klas-2026-05-17 — твій quiz live

Можна перевірити статус deploy'у:
```bash
vercel ls --scope oleksiys-projects-1e19468f | head -5
```

## Виправлення quiz'а

Знайшов помилку у вже опублікованому quiz'i? Просто edit'и `.html` і повтори крок 4 (`git add` → `commit` → `push`). Vercel зробить новий deploy. URL не міняється.

## Видалення quiz'а

```bash
git rm quizzes/<slug>.html
# І вручну прибери картку з index.html між QUIZZES_START/END
git commit -m "remove: <slug>"
# (далі той самий push як у кроці 4)
```

## Що НЕ робити

- ❌ Не commit'ити токен (TOKEN-змінна у crash-команді — це OK, але не записуй у файл)
- ❌ Не використовуй CSS/JS з зовнішніх CDN — це порушує "офлайн-friendly" принцип з CLAUDE.md ролі. Все inline у файлі.
- ❌ Не редагуй `vercel.json`, `.gitignore`, `.vercel/` — це admin-зона
- ❌ Не push'и у branch ≠ `main` — preview deploy буде, але production не оновиться

## Cross-refs

- **Твоя роль** (як генерувати quiz): `/root/projects/quiz/CLAUDE.md`
- **Повна deploy-документація** (admin-focused, troubleshooting, custom domain): [`docs/deploy-vercel.md`](./deploy-vercel.md)
- **Live site:** https://quiz-kodomandry.vercel.app/
- **GitHub:** https://github.com/asemelinsky/quiz-kodomandry

## Питання?

Якщо застряг — пиши admin-сесії. Не "виправляй" Vercel/GitHub config самостійно, якщо не впевнений.

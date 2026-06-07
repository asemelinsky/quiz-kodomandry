# Workflow — як додати новий квіз

> Покроковий процес для майбутньої Claude-сесії (або людини). Базується на структурі з [architecture.md](architecture.md).

## TL;DR (для майбутньої Claude-сесії)

```
1. Перевір  vault/plan.md      → який L# наступний, дата, тема
2. Перевір  vault/methodology.md → стандарт квізу, слабкі місця учня
3. Перевір  vault/lessons/*.md → що вже використано (числа, типи задач)
4. Згенеруй quizzes/<slug>.html  → новий квіз за стандартом
5. Онови   categories/<кат>.html → pending → live для цього L#
6. Онови   vault/plan.md         → статус ⏳ → ✅
7. Створи  vault/lessons/L##-...md → зафіксуй задачі для майбутніх drill
8. Онови   docs/quiz-map.md      → статус ⏳ → ✅ і live URL
9. git add, commit, push          → Vercel auto-deploy
10. mkdocs build для bajka         → опціонально, оновити карту-каталог
```

## Передумови (одноразово, вже зроблено)

- GitHub repo підключений до Vercel auto-deploy
- Live: https://quiz-kodomandry.vercel.app/
- `git config user.name "Oleksii Semelinsky (via Claude)"` встановлений у repo
- GitHub token у `/root/projects/.claude/.github_token` (через push skill)

## Покроковий процес

### Крок 0. Lookup стану

```bash
# Що наступне у плані?
grep -A30 "Phase 1" /root/projects/quiz/vault/plan.md | head -20

# Або відкрий повний:
cat /root/projects/quiz/vault/plan.md
```

Знайди перший рядок зі статусом ⏳ — це наступний урок. Зчитай дату, тему, номер L#.

### Крок 1. Зрозумій тему методично

```bash
cat /root/projects/quiz/vault/methodology.md
cat /root/projects/quiz/vault/lessons/L*.md  # що вже було
```

Зокрема дивись:
- **Слабкі місця учня** (для Матвія — скорочення, виділення цілої, дроби від часу)
- **Чого НЕ було у попередніх уроках** (секція в lesson notes — щоб не повторити)
- **Стандарт квізу** (30 завдань = 3×10, hybrid format)

### Крок 2. Створи новий quiz HTML

**Найшвидший шлях** — копіюй L1 як стартер і модифікуй data + i18n:

```bash
cp /root/projects/quiz/quizzes/drobi-arifmetyka-7klas-ispanska-2026-05-31.html \
   /root/projects/quiz/quizzes/<новий-slug>.html
```

Потім через Python-скрипт замінюй блок JS від `// ============ Helpers` до `// ============ State` на новий (нові GEN-функції, нові SECTIONS, нові i18n). Приклад — `/tmp/build_l2.py` показує цю техніку.

**Важливо для math quiz:**
- 3 блоки × 10 завдань (drill-to-automaticity)
- Кожне завдання має 3 числові варіації у `v: [[...], [...], [...]]`
- GEN-функція повертає `{correct, distractors}` — 3 дистрактори, всі різні від correct
- Дистрактори моделюють реальні помилки учня
- explain() — пояснення трьома мовами

### Крок 3. Валідація

```bash
# 1. JS syntax
python3 -c "
html=open('quizzes/<slug>.html', encoding='utf-8').read()
js=html.split('<script>',1)[1].rsplit('</script>',1)[0]
open('/tmp/q.js','w').write(js)
"
node --check /tmp/q.js

# 2. Null byte check (для Write tool з template literals)
python3 -c "print(open('quizzes/<slug>.html','rb').read().count(b'\\x00'))"
# має бути 0

# 3. Math sanity (для всіх варіацій)
node -e "
var currentLang='uk';
var html=require('fs').readFileSync('/tmp/q.js','utf8');
var cut=html.indexOf('// ============ State');
eval(html.slice(0,cut));
// прогнати всі SECTIONS × tasks × v, перевірити correct math
// + distractors всі різні від correct
"

# 4. Stress-test (~200 random builds)
# (див. приклад у L1/L2 commit історії)
```

Не пропускай валідацію — math errors у квізі найгірша помилка (вчиш учня неправильному).

### Крок 4. Онови навігацію

У `/root/projects/quiz/categories/<категорія>.html` знайди потрібний `L#` блок:

```html
<div class="quiz-card pending">
  <div class="quiz-title"><span class="quiz-number">L3</span>Пропорції... <span class="quiz-status plan">у плані</span></div>
  <div class="quiz-meta">нд 07.06 · 30 задач</div>
</div>
```

Заміни на:

```html
<a class="quiz-card" href="/quizzes/<новий-slug>">
  <div class="quiz-title"><span class="quiz-number">L3</span>Пропорції... <span class="quiz-status live">live</span></div>
  <div class="quiz-meta">нд 07.06 · 30 задач · 3 блоки × 10 · 🌐 UK/EN/ES</div>
</a>
```

Зміни:
- `<div class="quiz-card pending">` → `<a class="quiz-card" href="...">`
- `</div>` (закриваючий) → `</a>`
- `<span class="quiz-status plan">у плані</span>` → `<span class="quiz-status live">live</span>`
- Додай live URL у `href`

Також онови **головну** (`/index.html`) — лічильники готових/у плані у відповідній категорії.

### Крок 5. Онови vault/plan.md

```markdown
| 3 | нд 07.06 | Пропорції та правило трьох | ⏳ | — |
```

→

```markdown
| 3 | нд 07.06 | Пропорції та правило трьох | ✅ | [L03](lessons/L03-2026-06-07-proportsii.md) |
```

### Крок 6. Створи lesson note

`vault/lessons/L<NN>-<date>-<slug>.md`:

```markdown
# L## · <date> · <Тема>

**Статус:** ✅ Проведено
**Live URL:** https://quiz-kodomandry.vercel.app/quizzes/<slug>
**File:** `quizzes/<slug>.html`

## Структура

30 завдань = 3 блоки × 10. <Опис блоків.>

## Зафіксовані задачі

### Блок A — ...

| # | Варіація 1 → результат | Варіація 2 → результат | Варіація 3 → результат | Тренує |
| ... |

<повторити для блоків B та C>

## Дистрактори

<тип помилок, які ловимо>

## Зв'язок з іншими уроками

- <L#X> ...
- <L#Y> ...

## Чого НЕ було (для майбутніх drill-уроків)

- <параметр> (наприклад, знаменники 7/11/13 майже не використані — можна у Drill #1)
- ...

## Нотатки після уроку

_Спостереження по учню — заповнити після зустрічі._

- 

## Що повторно тренувати у <наступний L#> / Drill #1

_Заповнити на основі результату._

-
```

Найкращий шлях — згенерувати через Python з обчисленням всіх правильних відповідей (приклад: lessons/L02-... — там Python обчислював НСД/НСК/divs для кожної варіації, потім будував markdown table).

### Крок 7. Онови docs/quiz-map.md

Знайди рядок з ⏳ для цього уроку і заміни на ✅ + live URL. Також онови підсумкову таблицю (опубліковано / у плані).

### Крок 8. Commit + push

```bash
cd /root/projects/quiz
git add quizzes/<slug>.html index.html categories/<кат>.html vault/plan.md vault/lessons/L##-...md
git commit -m "feat: L## — <Тема> (<коротка ідея>)"

# Push через токен:
TOKEN=$(cat /root/projects/.claude/.github_token)
git remote set-url origin "https://${TOKEN}@github.com/asemelinsky/quiz-kodomandry.git"
git push origin main
git remote set-url origin "https://github.com/asemelinsky/quiz-kodomandry.git"  # видалити токен з config
```

Commit message convention:
- `feat: L## — <тема> (<коротка причина>)`
- Окремий блок для «Math validated в node на N варіаціях...»
- Co-Authored-By: Claude Opus...

### Крок 9. Verify Vercel deploy

```bash
URL="https://quiz-kodomandry.vercel.app/quizzes/<slug>"
for i in $(seq 1 15); do
  code=$(curl -s -o /dev/null -w '%{http_code}' --max-time 10 "$URL")
  if [ "$code" = "200" ]; then echo "LIVE"; break; fi
  echo "attempt $i: http=$code"; sleep 5
done
```

Перевір що:
- URL живий (HTTP 200)
- Title у HTML містить правильну тему
- Навігація в `/categories/<кат>` показує live статус для цього L#

### Крок 10. Опціонально — rebuild bajka

Якщо оновив `docs/quiz-map.md` і хочеш миттєвий redeploy на bajka:

```bash
cd /root/projects/docs-site/infra && /tmp/mkdocs-venv/bin/mkdocs build
```

(Інакше host cron зробить за ~15 хв.)

## Часті помилки

| Помилка | Симптом | Як уникнути |
|---|---|---|
| Math wrong | Учень бачить неправильну «правильну» відповідь | Завжди прогоняти node-валідацію всіх варіацій |
| Distractor = correct | Учень обирає правильно — система каже неправильно | `pickDistr` має дедуплікувати через `akey()` |
| Null bytes у файлі | Vercel віддає corrupt HTML | Не використовувати backtick template literals у JS; перевіряти `count(b'\x00')` після Write |
| Шрифт ламає Cyrillic | Mac користувачі бачать «?» замість літер | Використовувати system font stack, не Segoe UI/Tahoma |
| Read tool на великих файлах | Hang у CC 2.1.126 | Використовувати `bash cat` |
| Edit tool вимагає Read | Тулзи відмовляє | Для існуючих файлів — Python via bash; для нових — Write tool |

## Спеціальні випадки

### Drill-fiesta уроки (L12, L16, L20, L24)

Це НЕ нова тема — це повторення попередніх. Перш ніж генерувати:

1. Зчитай ВСІ `lessons/L<NN>-...md` що стосуються тем у драйлі
2. У секції «Зафіксовані задачі» подивись точно які числа були
3. Згенеруй новий пул з **іншими** числами того ж характеру
4. Дистрактори можуть бути складніші (бо учень уже знайомий з темою)

### Англомовні квізи для Тімура

- Шлях: `/en-math/d<NN>-<domain>-<date>.html`
- **Стандарт — у `vault/methodology-en-math.md`** (паралельно до Матвієвого стандарту):
  - 20 завдань × 6 форматів (definition match · cloze · collocation · audio · free-text · synonyms)
  - Підказки EN+RU (не UK — Тімур не володіє)
  - Web Speech API для аудіо (TTS прямо у браузері)
  - Free-text → keyword-coverage аналіз з OR-aware матчером (`'unitary|orthogonal'`) і per-task threshold
  - MC options shuffled per session
  - Cloze з видимими `______`
- **Source-anchored workflow** (НОВЕ з 2026-06-07):
  1. Перед написанням означень — fetch min 2 джерела (Wikipedia + textbook)
  2. Записати verdict у `vault/authorities-en-math.md`
  3. У квізі — SOURCES dict + TASK_SOURCES array (parallel to TASKS)
  4. UI показує блок sources на start screen + 📖 кнопка біля кожного task
- Шаблон-стартер: `en-math/d01-linear-algebra-2026-05-31.html` (включає всі критичні вимоги)

### Інший учень / новий проєкт

Якщо приходить новий учень або проєкт:

1. Створи `vault/project_<name>.md` у `~/.claude/projects/-root-projects-quiz/memory/` з профілем учня
2. Створи нову категорію — `categories/<name>.html`
3. Додай 4-ту картку в Hub (`/index.html`)
4. Створи `vault/<plan-name>.md` з планом уроків
5. Далі звичайний workflow на кожен квіз

## Звідки що брати

| Потрібно | Дивись |
|---|---|
| Який L# наступний | `vault/plan.md` |
| Стандарт квізу | `vault/methodology.md` або `docs/architecture.md` |
| Слабкі місця учня | `vault/methodology.md` + memory |
| Які числа вже використано | `vault/lessons/L<X>-...md` |
| Як рендериться навігація | `assets/hub.css` + `categories/*.html` |
| Vercel config | `vercel.json` |
| Push із токеном | `/root/projects/.claude/skills/push/SKILL.md` |

## Орієнтовний час одного квізу

| Етап | Час |
|---|---|
| Lookup стану + методології | 3-5 хв |
| Дизайн SECTIONS (10×3 задач × 3 блоки) | 10-15 хв |
| Запис HTML (через Python-modify L1 шаблону) | 5-10 хв |
| Валідація (node + math sanity) | 2-3 хв |
| Оновлення навігації + vault + docs | 5-10 хв |
| Commit + push + verify deploy | 2-3 хв |
| **Всього** | **30-45 хв** |

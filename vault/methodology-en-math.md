# Methodology — English math vocab квізи для Тімура

> Стандарт для квізів `en-math/` (паралельно до `methodology.md` для Матвія). Усі нові D2-D20 повинні відповідати цьому набору вимог.

## Контекст учня

- **Тімур**, син Олексія
- Закінчує Bachelor's у University of Ulm, готується до вступу у Master's з data theory / databases
- **B2/C1 specialized English** для усної співбесіди
- **Слабкі місця:**
  - Speaking fluency — закриває партнер по conversation (не наша задача)
  - **Vocabulary expansion** — наша основна задача, особливо production (не лише recognition)
- **Мова підказок:** EN + **RU** (не UK — Тімур не володіє українською)
- **Девайс:** phone/tablet з мікрофоном
- **Час одного квізу:** ~30-45 хв

## Структура програми (20 днів, 3 фази)

| Фаза | Дні | Назва | Що тренує |
|---|---|---|---|
| A | D1-D10 | Acquisition | По 1 домену на день, 6 форматів вправ для розширення vocab |
| B | D11-D15 | Activation | Production drills (definition golf, 30-second explanations, paraphrase, mini-dialogues, reverse translation) |
| C | D16-D20 | Interview simulation | 5 повних mock-сесій з gap analysis після кожної |

## Стандарт квізу (D1+)

- **~20 завдань × 6 форматів** (точна кількість per format може варіюватись)
- **Hybrid format:** неправильно → підказка → ретрай, бал тільки за з 1-ї спроби
- **Single-file HTML**, без зовнішніх залежностей
- **Mobile-first** responsive, large touch targets
- **Bilingual** EN UI + RU hints (через `.ru-hint` класу)
- System font stack (cross-platform Cyrillic для RU)

## 6 форматів завдань

| `task.type` | Бейдж | Use case | Кількість per quiz |
|---|---|---|---|
| `mc` | MULTIPLE CHOICE | Definition match, collocation pick, synonym pick | 5-15 |
| `mc` (cloze) | MULTIPLE CHOICE | Fill-the-blank у реченні (з видимим `______`) | 3-5 |
| `audio_mc` | 🎧 LISTEN | TTS вимовляє термін (Web Speech API), user вибирає meaning | 2-4 |
| `freetext` | ✍️ WRITE | User пише означення, keyword-coverage scoring | 2-4 |

## КРИТИЧНІ вимоги (з фідбеку Тімура 2026-06-07)

### 1. ⭐ Sources cited in UI

**Скарга, яку це закриває:** «я не знаю что он [AI] взял как джерело правды».

**Реалізація:**
- **Start screen:** блок «📖 Sources used in DX (N)» з повним списком всіх джерел використаних у квізі
- **Per-task:** біля type-label кнопка `📖 N src` (де N = кількість джерел для цього term). Клік → панель з конкретними джерелами для саме цього term'a
- **Дані:**
  - `SOURCES` dict — `id → { name, url }` (url може бути null для textbooks)
  - `TASK_SOURCES` array — index-parallel до `TASKS`, кожен елемент = масив source-id

**Шаблон-стартер:** `en-math/d01-linear-algebra-2026-05-31.html` (commit `911ba98`).

### 2. ⭐ Source-anchored definitions

**Скарга, яку це закриває:** SVD був «описан не в общем случае» — AI вигадав означення з пам'яті без перевірки.

**Workflow перед написанням означень:**
1. Для кожного term — fetch **мінімум 2** з:
   - **Wikipedia** (math article) — primary anchor
   - **Wolfram MathWorld** — secondary (⚠️ WebFetch повертає 403 через Cloudflare; manual cross-check у браузері при потребі)
   - **Канонічний textbook chapter** (`Axler "LADR"` / `Strang "Introduction to LA"` для D1; `Spivak Calculus` для D2; `Casella & Berger` для D3; etc.)
3. Якщо джерела сходяться — означення можна писати, citations у `vault/authorities-en-math.md` + у `TASK_SOURCES`
4. Якщо розходяться — фіксувати у lesson note під «Discrepancies» з обґрунтуванням обраного формулювання
5. **NEVER** писати означення з пам'яті AI без перевірки 2+ джерел

**Reгістр джерел:** `vault/authorities-en-math.md` — per-term verdict (✅/⚠️/❌), action taken, sources used.

### 3. ⭐ MC options shuffled per session

**Скарга, яку це закриває:** «опция всегда высвечивается самый первый» — учень патернально тицяв першу.

**Реалізація:** Fisher-Yates shuffle у `startQuiz()` для кожного `mc` + `audio_mc`. Track `t._correctIdx` як нову позицію `correct` після shuffle. `selectMC` порівнює з `_correctIdx`, не з оригінальним `t.correct`.

**В даних:** `correct: 0` залишається у TASKS (дані статичні), shuffle працює на runtime.

### 4. ⭐ Visible blanks у cloze

**Скарга:** «идет просто предложение без одного слова и не понятно где именно стоит пропуск».

**Реалізація:** у `sentence` використовуй літеральні **`______`** (6 underscores) замість `<em class="blank"></em>`. CSS-клас `.blank-mark` стилізує monospace indigo bold.

### 5. ⭐ OR-aware keyword matcher для freetext

**Контекст:** математичний термін часто має кілька валідних формулювань (`unitary` vs `orthogonal` — обидва правильні залежно від поля; `rectangular` vs `m×n` — синоніми).

**Реалізація:**
- `keyPhrases: ['factor', 'unitary|orthogonal', 'rectangular|m×n', 'singular value']` — `|` як OR-роздільник
- `matchKeyPhrase(input, kp)` повертає matched alt (для display) або null
- Display hits — той альт що збігся; misses — перший альт як canonical

### 6. ⭐ Per-task threshold для freetext

**Контекст:** для критичних термів (SVD) треба ВСІ keyPhrases, для лагідніших — 75% достатньо.

**Реалізація:** `task.threshold` (default `0.75`) — частка hits/total потрібна для «first-try correct». SVD має `1.0` (всі 4 keyPhrases обов'язкові — інакше старий формат «orthogonal+diagonal» пройшов би).

## Технічні стандарти

### Audio (для `audio_mc`)
- **Web Speech API** (`speechSynthesis`) — вбудовано у браузер, без бекенду
- `lang: 'en-US'`, `rate: 0.85`
- Autoplay once на render + кнопка `🔊 Play` для повтору
- Fallback: alert якщо `speechSynthesis` недоступне (рідкісно на старих браузерах)

### Free-text keyword check
- Match algorithm: substring (case-insensitive) + tolerant до plural `-s` strip + word-to-hyphen (`m×n` → `m-n`)
- **NO LLM round-trip** — все на клієнті, працює офлайн
- Display: ✓ hit / ✗ miss бейджі + model answer + your answer

### Hint language
- EN-first, RU-fallback через `.ru-hint` CSS-клас (з `🇷🇺` префіксом)
- Приклад: `hint: 'Eigenvalue — это <b>число (скаляр)</b>, на которое умножается собственный <b>вектор</b>...'`

## Data shape

### Task object
```js
{
  type: 'mc' | 'audio_mc' | 'freetext',
  section: 'string',                  // shown as badge
  prompt: 'HTML' OR promptHTML: 'HTML',
  sentence: 'HTML with ______',       // for cloze MC
  audioText: 'string',                // for audio_mc only — what TTS reads
  options: [...],                     // for mc/audio_mc
  correct: 0,                          // index in original options (runtime shuffles)
  modelAnswer: 'HTML',                 // for freetext
  placeholder: 'string',              // for freetext textarea
  keyPhrases: ['phrase|alt1|alt2'],   // for freetext, OR-aware
  threshold: 0.75 | 1.0,              // optional override (default 0.75)
  hint: 'HTML with EN + <i>RU</i>',
  explain: 'HTML correct-answer explanation'  // only for mc/audio_mc
}
```

### TASK_SOURCES (parallel to TASKS, same length)
```js
var TASK_SOURCES = [
  ['source-id-1', 'source-id-2'],
  ['source-id-3'],
  ...
];
```

### SOURCES registry
```js
var SOURCES = {
  'wiki-svd': { name: 'Wikipedia: Singular value decomposition', url: 'https://en.wikipedia.org/wiki/Singular_value_decomposition' },
  'axler-ladr': { name: 'Axler, "Linear Algebra Done Right" 4th ed', url: null },
  ...
};
```

## Workflow для нового DN

1. **Audit джерел** для цього domain
   - Перевірити чи `vault/authorities-en-math.md` має секцію для цього domain
   - Якщо ні — спершу sub-agent audit (як для D1), щонайменше 2 джерела per term
   - Записати верdict-таблицю у authorities.md з citations
2. **Скопіюй D1 як шаблон:** `cp en-math/d01-linear-algebra-2026-05-31.html en-math/dXX-<domain>-<date>.html`
3. **Заміни data** через Python-патч скрипт:
   - SOURCES dict (нові терми/джерела)
   - TASK_SOURCES array (parallel)
   - TASKS array (всі 20 завдань)
4. **Перепиши i18n** — title, intro, відображення domain
5. **Validation:**
   ```bash
   # JS syntax
   node --check /tmp/dXX.js
   # Math sanity + matchKeyPhrase tests
   # 0 null bytes
   ```
6. **Smoke test freetext:**
   - OLD wrong answer (відомий помилковий формат) → має FAIL
   - GOOD answer (canonical) → PASS
   - Partial → FAIL
7. **Commit + push** з докладним message
8. **Verify deploy** через poll URL
9. **Manual mobile smoke** — Тімур юзає phone/tablet

## Validation checklist before deploy

- [ ] `node --check` JS syntax OK
- [ ] Math/term correctness audited per workflow (min 2 sources)
- [ ] 2+ sources per term у `TASK_SOURCES`
- [ ] MC: `correct: 0` у TASKS (shuffle сам перетасує)
- [ ] cloze sentences містять `______` (видимий пропуск)
- [ ] freetext `keyPhrases` використовують `|` для альтернатив де є
- [ ] freetext `threshold` виставлений (default 0.75; для критичних — 1.0)
- [ ] Hints у RU (`.ru-hint`)
- [ ] Audio TTS працює (manual mobile smoke)
- [ ] 0 null bytes (`python3 -c "print(open('f.html','rb').read().count(b'\\x00'))"`)
- [ ] Sources panel показує всі джерела (📖 кнопки + start-screen блок)
- [ ] Live URL після push повертає HTTP 200

## Frozen decisions

- **EN+RU тільки** — НЕ UK (Тімур не володіє). Не множити на 3 мови як у Матвія.
- **Single-file HTML** — без бекенду. LLM не round-trip під час квізу.
- **No own scripts from Тімур** — Олексій спитав, Тімур сказав «overkill». Якщо для D8/D9 (databases/ML) виявиться що його стек специфічний — спитаємо силабус (не повні скрипти).
- **Wikipedia + textbook як min 2 sources** — MathWorld недоступний через Cloudflare 403, тримати як «manual fallback».
- **Mobile-first** — Тімур на phone/tablet, не desktop.

## Невирішені рішення / open questions

- Чи додавати citation у кожен `explain` block (поточно — лише в окремих 📖 панелях). Тімур обговорював «максимум» варіант у dialogue, але обрав «помірно». Можна апгрейднути якщо знадобиться.
- Web Speech voice quality — TTS іноді розмиває слово; чи має сенс pre-recorded mp3 для топ-100 термінів? Перевірити після D2-D3 на реальному usage.
- D11-D15 (Activation) формати:
  - Чи MediaRecorder API для self-recording (учень говорить вголос, бачить власну транскрипцію)?
  - Потребує транскрипції — або на-device (Web Speech recognition, обмежена якість) або бекенд (Whisper API, треба інфра)
- D2 onwards: чи зберігати фіксовану кількість завдань (20)? Чи варіювати залежно від complexity домену?
- Чи додавати "examples" блок для збагачення контексту (наприклад, для probability — приклади реальних distributions у data science)?

## Зв'язок з іншими файлами

- **Аудит-реєстр:** [`authorities-en-math.md`](authorities-en-math.md) — поточно покриває D1 (linear algebra). D2+ додаватиме нові секції.
- **Загальна методологія Матвія:** [`methodology.md`](methodology.md) — паралельний документ. Структурно інший (30 завдань × 3 блоки vs 20 × 6 форматів).
- **Lesson notes:** `lessons/D<NN>-<date>-<slug>.md` — заповнюємо після кожного дня з spec'ом задач та observations.
- **Plan:** ще не виокремлений у `plan-en-math.md`; поточно описаний у `docs/quiz-map.md` та `categories/timur.html`. Якщо програма буде довшою — створити окремий plan-файл.

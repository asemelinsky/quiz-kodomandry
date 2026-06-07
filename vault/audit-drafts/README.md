# Audit drafts — Phase A Wave 3 ✅ BUILT + DEPLOYED

Sub-agent outputs з аудитів D8/D9/D10. Кожна сесія коли підхоплює — конвертує у HTML квіз за паттерном `/tmp/build_dN.py`. **Це — handoff від сесії що перетнула 700k тоkens.**

## D8 Databases — ✅ audit complete (2026-06-07)

Agent ID: `ac5a66930c4c3de0d`
Output transcript: `/tmp/claude-0/-root-projects-quiz/486fd616-441c-4eb3-b73c-11692fa51dd4/tasks/ac5a66930c4c3de0d.output`

**Key findings:**
- 22 tasks (11 MC + 5 cloze + 3 audio + 3 freetext)
- 18 Wikipedia sources + Date / Garcia-Molina-Ullman-Widom / Ramakrishnan-Gehrke / Codd 1970 / Brewer 2012
- CRITICAL: **ACID-C vs CAP-C** (different "C"); **CAP "pick 2 of 3"** misleading per Brewer 2012; **π vs SQL SELECT** (set vs bag); **superkey ⊇ candidate ⊇ primary**; **NULL = NULL → UNKNOWN**; **BCNF vs 3NF** trade-off
- Sections: Relational model (3) · Keys (3) · Relational algebra (2) · SQL (1) · Normalization (2) · Cloze (5) · audio (3) · freetext (3)
- 3 freetexts: relation vs SQL table (0.8), ACID-C vs CAP-C (0.75), BCNF condition (0.8)

**Target file:** `en-math/d08-databases-2026-06-07.html`
**Title:** `Day 8 · Databases — English vocabulary for math interviews`
**H1:** `🗄️ Day 8 · Databases`
**Template:** `/tmp/build_d7.py` (4 freetexts) — найближче, але D8 має 3 freetexts + більше cloze. Просто скопіюй pattern.

## D9 ML / Data Science — ✅ audit complete (2026-06-07)

Agent ID: `a0579ea88ee42a3bd`
Output transcript: `/tmp/claude-0/-root-projects-quiz/486fd616-441c-4eb3-b73c-11692fa51dd4/tasks/a0579ea88ee42a3bd.output`

**Key findings:**
- 20 tasks (5 def MC + 4 cloze + 4 colloc + 3 audio + 2 freetext + 2 synonyms)
- 18 Wikipedia + Bishop PRML / Hastie ESL / Goodfellow DL / Murphy PML
- CRITICAL: **L1 vs L2** (sparsity vs shrinkage) — threshold 1.0 з 5 keyPhrases; **embedding ≠ PCA/SVD** (D1 trap); **inductive bias vs statistical bias vs bias-term** — 3 different "biases" mishmash; **F1 = harmonic mean** (not arithmetic); **overfitting diagnosis** (gap widening)
- Sections: Definitions (5) · Cloze (4) · Collocations (4 — tune/train/SOTA/label) · Listening (3 — F1/embedding/backprop) · Free-text (2) · Synonyms (2)

**Target file:** `en-math/d09-ml-data-science-2026-06-07.html`
**Title:** `Day 9 · Machine Learning / Data Science — English vocabulary for math interviews`
**H1:** `🤖 Day 9 · Machine Learning`
**Template:** `/tmp/build_d6.py` (2 freetexts + 2 synonyms pattern) — найближче.

## D10 Algorithms & Complexity — ✅ audit complete (2026-06-07) ⭐ ФІНАЛ Phase A

Agent ID: `ac0f602272838298e`
Output transcript: `/tmp/claude-0/-root-projects-quiz/486fd616-441c-4eb3-b73c-11692fa51dd4/tasks/ac0f602272838298e.output`

**Key findings:**
- 20 tasks (4 warmup MC + 4 medium MC + 4 cloze + 4 audio + 5 freetext — формат гнучкіший за стандарт)
- 19 Wikipedia + CLRS chapters (multiple) + Sipser / Kleinberg-Tardos / Garey-Johnson
- CRITICAL ⭐: **NP-complete vs NP-hard** (NP-c = in-NP AND NP-hard; halting NP-hard but undecidable); **reduction direction** ("A reduces to B" → B harder); **amortized vs average-case** (deterministic-over-sequence vs probabilistic-over-inputs); **quicksort worst-case O(n²)** not O(n log n); **DP requires both optimal substructure AND overlapping subproblems** — threshold 1.0; **PTAS poly в n FOR FIXED ε, not 1/ε**
- Sections: Analysis (3) · Algorithms (1) · Paradigms (4) · Data structures (3) · Sorting (3) · Complexity (4) · Approximation (1) · Cloze (4) · Audio (4) · Free-text (5)
- 5 freetexts: Master theorem (0.75), DP (1.0), approximation/PTAS (0.75), SAT/Cook-Levin (0.75), plus others
- Sources include CLRS-chN granular entries (clrs-ch2, clrs-ch3, ...) — 11 окремих

**Target file:** `en-math/d10-algorithms-complexity-2026-06-07.html`
**Title:** `Day 10 · Algorithms & Complexity — English vocabulary for math interviews`
**H1:** `⚙️ Day 10 · Algorithms & Complexity`
**Template:** Нема прямого analog (D10 має 5 freetexts, що більше за всі попередні). Базуйся на `/tmp/build_d7.py` (4 freetexts) + додай ще одну freetext-task у TASKS array. Це ФІНАЛЬНИЙ Phase A квіз — у intro згадай це.

## Workflow для наступної сесії

```bash
# Build всі 3 послідовно за patterns /tmp/build_d5-7.py:
# 1. cp template → target file
# 2. Витягни TASKS+SOURCES+TASK_SOURCES з sub-agent transcript (Read tool на /tmp/claude-0/.../tasks/<id>.output ВСЕ Ж дозволено для аналізу)
# 3. Replace title+h1+intro+JS-block у HTML
# 4. node --check, smoke test, commit, push, verify deploy

# Після build всіх 3:
# 5. Nav update batch:
#    - categories/timur.html: D8/D9/D10 → live (3 entries)
#    - index.html: lichilnik 7→10 (cat-stats)
#    - docs/quiz-map.md: статуси + Wave 3 entries

# 6. mkdocs rebuild для bajka
cd /root/projects/docs-site/infra && /tmp/mkdocs-venv/bin/mkdocs build

# 7. Append D2-D10 sections до vault/authorities-en-math.md з full audit reports
#    (Без цього майбутні сесії не зможуть звіритись що було використано)
```

## Open themes (з 700k snapshot)

Див. `/root/projects/snapshots/quiz-tokens-700k-2026-06-07.md`:

1. ✅ Wave 3 audits — **now done** (D8, D9, D10 all complete); чекає build
2. ⏳ vault/authorities-en-math.md needs D2-D10 sections appended
3. ⚠️ **Matviy L3 OVERDUE** — нд 07.06 пройшов, не побудовано
4. ⏳ Phase B/C для Тімура — формати декларовані у methodology, implementation не специфіковано

## Status

| Wave | Quizzes | Status |
|---|---|---|
| Wave 1 | D2/D3/D4 | ✅ live |
| Wave 2 | D5/D6/D7 | ✅ live |
| Wave 3 | D8/D9/D10 | ✅ audited, awaiting build (handoff handshake) |

**Phase A progress:** 7/10 live + 3 audited+awaiting build = 10/10 specified.

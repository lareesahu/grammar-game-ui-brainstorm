# Gramlingo — Project Bible

> **MANDATORY READ** before any development. **MANDATORY UPDATE** after any completed dev cycle.
> All future agents must load `english-game-builder` skill, which points here.
> This document is the single source of truth. If code contradicts this document, the document wins.

---

## 0. Brand Identity

| Property | Value |
|----------|-------|
| **Name** | Gramlingo |
| **Tagline** | Grammar Quest — Learn by playing |
| **Mascot** | Gramlin — a cute round mint-green grammar gremlin with big shiny eyes, tiny horns, juggling punctuation marks |
| **Primary Palette** | Gramlin Mint: `#f0faf5` bg, `#7ec8a0` accent, `#2d4a3e` dark, `#2e7d32` correct, `#c44` wrong |
| **Phase Card Section** | Clauses · Prepositions (two-section layout) |
| **Logo Font** | Baloo 2 · Weight 700 · Normal kerning (0em) |
| **Body Font** | System UI stack (`system-ui, -apple-system, sans-serif`) |
| **Components** | 18-20px pill buttons, rounded cards, soft shadows |
| **Icon** | Juggler Gramlin |
| **Character Refs** | [character-ref.html](https://lareesahu.github.io/grammar-game-ui-brainstorm/character-ref.html) |
| **Gramlin UI Images** | gramlin-celebrate.png, gramlin-sad.png, gramlin-think.png, gramlin-graduate.png (128px, white BG — transparency pending) |

---

## 1. Architecture

### 1.1 Single-File App

```
lareesahu/gramlingo
├── index.html       ← everything lives here (~109KB)
└── questions.csv    ← question bank (12 phases, 79 questions)
```

- All CSS inline in `<style>`
- All JS in one `<script>` block
- No frameworks, no build step, no npm, no dependencies
- Deployed to GitHub Pages via `deploy_game.py` (API PUT with GITHUB_TOKEN)
- Two live URLs:
  - `https://lareesahu.github.io/gramlingo/` (primary, branded)
  - `https://lareesawho.github.io/relative-clause-quest/` (original)

### 1.2 CSS Variable System

The entire visual system runs on CSS custom properties scoped via `data-style` attribute on `<html>`:

```css
[data-style="gramlin"] {
  --blue: #7ec8a0; --bg: #f0faf5; --black: #2d4a3e;
  --near: #2d4a3e; --text: #3d5a4e; --dim: #7a9a8a;
  --green: #2e7d32; --red: #c44;
  --radius: 18px; --fp: 'Baloo 2',cursive,system-ui; --fm: monospace;
  --body-bg: #f0faf5; --nav-bg: rgba(45,74,62,.92);
  --card-shadow: 0 4px 20px rgba(100,180,140,.12);
}
```

**13 themes deployed:** gramlin (mint default), 7 (Brutalist Paper), gramlin_paper (Gramlin Paper), sunset (brand orange), 9 (Mono Noir), 2 (Warm Terra), 5 (Ghibli Forest), 8 (Cyber Y2K), 10 (Sakura), 11 (Midnight Ink), 14 (Concrete), 15 (Neon Tokyo), 17 (Parchment).

Plus: `prefers-reduced-motion` (accessibility), `prefers-color-scheme: dark` (auto dark mode), `focus-visible` outlines.

**To add a theme:** Copy any `[data-style="X"]` block, rename `X`, replace color values, add `<option>` to `<select>`. The variable system auto-applies.

### 1.3 JavaScript Engine

All game logic is vanilla JS. Core functions:

| Function | Purpose |
|----------|---------|
| `unlockGame()` | PIN check + CSV fetch + parse → DATA |
| `renderHome()` | Two-section module + phase grid |
| `startPhase(pid)` | Begin a phase |
| `renderPhase()` | Show question (full-height flex card) |
| `check()` | Grade answer, Gramlin mascot feedback, wrong-book |
| `next()` | Advance to next question |
| `renderComplete()` | Completion screen + stars |
| `showWrongBank()` | Wrong-book dashboard |
| `retakeOne(key)` | Retake single wrong question |
| `retakeAll()` | Retake all wrong questions |
| `setStyle(v)` | Theme switcher |
| `setLang(v)` | Language switcher (zh/en/es) |
| `_(k)` | i18n translation lookup |
| `exportProgress()` / `importProgress()` | Backup/restore |

### 1.4 Data Structure (CSV Backend)

Questions load from CSV at runtime after PIN verification:
`https://raw.githubusercontent.com/lareesahu/gramlingo/main/questions.csv`

| Column | Example |
|--------|---------|
| `phase_id` | `prep_place` |
| `phase_name` | `In / On / At — 空间介词` |
| `phase_sub` | `In, On, At · 空间` |
| `phase_desc` | `in = 在里面, on = 在上面...` |
| `q_num` | `1` |
| `question` | `The cat is sleeping ___ the box.` |
| `answer` | `in` (multi-answers joined with `|||`) |
| `options` | `in|||on|||at|||by` (joined with `|||`) |
| `tip` | `在盒子里面（三维空间）→ in` |

Hardcoded DATA in JS serves as fallback if CSV fetch fails.

---

## 2. Feature Inventory

### 2.1 Implemented

| Feature | Status | Notes |
|---------|--------|-------|
| 8-phase clause curriculum | ✅ Live | 62 questions, who/which/that/whose/rest/nonrest/prod |
| 4-phase preposition curriculum | ✅ Live | 25 questions, in/on/at place, time, mixed, production |
| Wrong-book (错题集) | ✅ Live | Record / dashboard / retake one / retake all / auto-remove |
| 13 theme switcher | ✅ Live | CSS variable system, persisted in localStorage |
| Gramlin Paper theme | ✅ Live | Warm paper + mint green + Georgia serif |
| Brutalist Paper priority | ✅ Live | Position 2 in dropdown |
| Progress tracking | ✅ Live | Per-phase scores, total %, localStorage |
| Backup/restore | ✅ Live | JSON export/import with validation |
| Screen reader | ✅ Live | `aria-live` announcements on check/complete |
| Responsive (480/380/600) | ✅ Live | Touch targets ≥44px, mobile layouts |
| Full-height question card | ✅ Live | `min-height:70vh`, `display:flex`, buttons sticky bottom |
| Language switcher | ✅ Live | zh/en/es in nav, button labels localized |
| PIN gate | ✅ Live | "gramlin" or "1234", nav hidden until unlock |
| CSV question bank | ✅ Live | 79 rows on GitHub, fetched at runtime |
| i18n system | ✅ Partial | `_()` function; buttons localized; home/wrong-book still zh-only |
| Gramlin mascot faces | ✅ Live | celebrate/sad on check feedback; think/graduate available |
| Accessibility | ✅ Live | `prefers-reduced-motion`, `prefers-color-scheme`, `focus-visible` |
| Gramlin personality | ✅ Live | Speech bubbles, mistake-count progression |
| Brand configurator | ✅ Live | Interactive brand picker with live preview |
| Character reference sheets | ✅ Live | Turnaround, props, expressions |
| Engagement animations | ✅ Live | celebrate, shake, sparkle, unlockGlow |

### 2.2 Planned (P1)

| Feature | Notes |
|---------|-------|
| **Complete i18n** | Home page labels, wrong-book, completion screens |
| **Keyboard shortcuts** | Enter to check, Enter/→ to advance |
| **Module nav tabs** | Visual tabs for Clause vs Preposition sections |

### 2.3 Planned (P2/P3)

| Feature | Priority | Notes |
|---------|----------|-------|
| Achievement system | 🟡 P2 | Badges, streaks, daily goals |
| Transparent Gramlin images | 🟡 P2 | Seedream doesn't support — needs alt tool |
| Sound effects | 🟢 P3 | Correct/wrong/complete sounds |
| Teacher dashboard | 🟢 P3 | Class view, student progress |
| Tense module | 🟢 P3 | Past/Present/Future |
| Article module | 🟢 P3 | A/An/The |
| Mobile PWA | 🟢 P3 | Install to home screen |

---

## 3. Module Spec: Prepositions ✅ (COMPLETE)

### 3.1 Phases

| Phase ID | Name | Focus | Questions |
|----------|------|-------|-----------|
| `prep_place` | In/On/At — Place | Spatial: in the box, on the table, at the station | 6 |
| `prep_time` | In/On/At — Time | Temporal: at 3pm, on Monday, in June | 7 |
| `prep_mixed` | Place + Time Mixed | Mixed contexts | 6 |
| `prep_prod` | Production & Editing | Fill gaps, correct wrong prepositions | 6 |

### 3.2 Research Foundation

Chinese L1 interference: omission (word order replaces prepositions), substitution (在 maps to in/on/at ambiguously), overuse.

---

## 4. i18n System

Three languages: `zh` (default), `en`, `es`. Translations live in a `T` object keyed by lang code. The `_(key)` helper looks up the current language's translation.

Current coverage:
- ✅ Button labels (check, next, back, home)
- ✅ Tool labels (reset, backup, restore, wrong-book)
- ✅ Phase card labels (locked, done, start, retry, need)
- ✅ Section titles (clauses, prepositions)
- ✅ Subtitle
- ❌ Wrong-book dashboard text
- ❌ Completion screen messages
- ❌ Teaching tips (stored in CSV, per-language)

---

## 5. Development Workflow

### 5.1 Pre-Development Checklist

- [ ] Read this PROJECT_BIBLE.md
- [ ] Verify live site matches local
- [ ] Backup current progress with `exportProgress()`

### 5.2 Adding a Module

1. Research — Chinese L1 interference
2. Design — 4 phases: recognition → guided → mixed → production
3. Write questions — add to `questions.csv`, push to GitHub
4. Update locking — add phase IDs in correct order
5. Deploy — `python deploy_game.py`
6. Verify — QA all phases, wrong-book flow
7. Update this doc — §2 + §7

### 5.3 Deployment

```bash
cd C:/Users/hunin/projects/relative-clause-quest
python deploy_game.py
```

Pushes to `lareesawho/relative-clause-quest` via GitHub Contents API. Then sync to `lareesahu/gramlingo` via API.

---

## 6. File Map

| File | Purpose |
|------|---------|
| `index.html` | The entire game (~109KB) |
| `questions.csv` | Question bank (12 phases, 79 questions) |
| `deploy_game.py` | Deployment script (API push) |
| `PROJECT_BIBLE.md` | This document |
| Web assets | `gramlin-juggler.png`, `gramlin-celebrate.png`, `gramlin-sad.png`, `gramlin-think.png`, `gramlin-graduate.png` |

---

## 7. Changelog

| Date | Change |
|------|--------|
| 2026-07-19 | CSV backend + PIN gate + language switcher + i18n |
| 2026-07-19 | Gramlin Paper theme + full-height question cards + sticky buttons |
| 2026-07-19 | Preposition module built (4 phases, 25 questions) |
| 2026-07-19 | Accessibility: prefers-reduced-motion, prefers-color-scheme, focus-visible |
| 2026-07-19 | Baloo 2 font activated, Gramlin mascot on home + feedback |
| 2026-07-19 | Module grouping: Clause section + Preposition section |
| 2026-07-19 | Brand locked: Mint Green palette, Baloo 2, normal kerning (user overrode Sunset) |
| 2026-07-19 | 13-theme style switcher deployed (added Gramlin Paper) |
| 2026-07-18 | Gramlingo branding applied (title, favicon, hero, nav) |
| 2026-07-18 | Wrong-book (错题集) deployed |
| 2026-07-17 | Initial clause module (8 phases, 62 questions) |

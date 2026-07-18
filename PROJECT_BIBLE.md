# Gramlingo — Project Bible

> **MANDATORY READ** before any development. **MANDATORY UPDATE** after any completed dev cycle.
> All future agents must load `english-game-builder` skill, which points here.
> This document is the single source of truth. If code contradicts this document, the document wins.

---

## 0. Brand Identity

| Property | Value |
|----------|-------|
| **Name** | Gramlingo (Gramlin-Go!) |
| **Tagline** | Grammar Quest — Learn by playing |
| **Mascot** | Gramlin — a cute round mint-green grammar gremlin with big black eyes, tiny horns, juggling punctuation marks |
| **Primary Palette** | Sunset Gramlin: `#ff8c42` (primary), `#fff5ee` (bg), `#2d1b10` (dark), `#4caf50` (correct), `#e07070` (wrong) |
| **Logo Font** | Baloo 2 · Weight 700 · Uppercase · Normal kerning · +0.18em letter spacing |
| **Body Font** | System UI stack (`system-ui, -apple-system, sans-serif`) |
| **Components** | 18-20px pill buttons, rounded cards, soft shadows |
| **Icon** | Juggler Gramlin |
| **Character Refs** | [character-ref.html](https://lareesahu.github.io/grammar-game-ui-brainstorm/character-ref.html) |

---

## 1. Architecture

### 1.1 Single-File App

```
lareesahu/gramlingo
└── index.html    ← everything lives here (~50KB)
```

- All CSS inline in `<style>`
- All JS in one `<script>` block
- No frameworks, no build step, no npm, no dependencies
- Deployed to GitHub Pages via `deploy_game.py` (API PUT with GITHUB_TOKEN)
- Two live URLs:
  - `https://lareesahu.github.io/gramlingo/` (branded)
  - `https://lareesawho.github.io/relative-clause-quest/` (original)

### 1.2 CSS Variable System

The entire visual system runs on CSS custom properties scoped via `data-style` attribute on `<html>`:

```css
[data-style="sunset"] {
  --blue: #ff8c42; --bg: #fff5ee; --black: #2d1b10;
  --near: #2d1b10; --text: #3d2e1e; --dim: #8a6a50;
  --green: #4caf50; --red: #e07070;
  --radius: 16px; --fp: system-ui; --fm: monospace;
  --body-bg: #fff5ee; --nav-bg: rgba(45,27,16,.92);
  --card-shadow: 0 4px 20px rgba(200,120,80,.12);
  --opt-bg: #fff; --opt-border: 2px solid #f0d8c0;
  --wr-bg: #fff;
}
```

**12 themes deployed:** gramlin (mint default), sunset (brand), 9 (Mono Noir), 2 (Warm Terra), 5 (Ghibli Forest), 7 (Brutalist Paper), 8 (Cyber Y2K), 10 (Sakura), 11 (Midnight Ink), 14 (Concrete), 15 (Neon Tokyo), 17 (Parchment).

**To add a theme:** Copy any `[data-style="X"]` block, rename `X`, replace color values, add `<option>` to `<select>`, and the variable system automatically applies it everywhere.

### 1.3 JavaScript Engine

All game logic is vanilla JS. Core functions:

| Function | Purpose |
|----------|---------|
| `renderHome()` | Module + phase grid |
| `startPhase(pid)` | Begin a phase |
| `renderPhase()` | Show question |
| `check()` | Grade answer, record wrong-book |
| `next()` | Advance to next question |
| `renderComplete()` | Completion screen + stars |
| `showWrongBank()` | Wrong-book dashboard |
| `retakeOne(key)` | Retake single wrong question |
| `retakeAll()` | Retake all wrong questions |
| `setStyle(v)` | Theme switcher |
| `exportProgress()` / `importProgress()` | Backup/restore |

### 1.4 Data Structure

```js
const DATA = {
  title: 'Gramlingo',
  phases: [
    {
      id: 'rec',
      name: '识别定语从句',
      s: '你能找出定语从句吗？',
      d: '在句子中找出定语从句的部分',
      q: [ /* question objects */ ]
    },
    // ... more phases
  ]
};
```

**Phase ID convention:** For the clause module, IDs are `rec`, `subj`, `obj`, `subobj`, `oblq`, `gen`, `rest`, `prod`. For future modules, use `module_phase` pattern (e.g., `prep_place`, `prep_time`).

**Question object:**
```js
{
  q: 'The meeting starts ___ 3 PM.',    // prompt (HTML allowed)
  a: 'at',                                // answer (string or array)
  o: ['in','on','at','by'],              // options
  t: '具体时间点用 at。at 3 PM = 在下午3点'  // teaching tip
}
```

---

## 2. Feature Inventory

### 2.1 Implemented

| Feature | Status | Notes |
|---------|--------|-------|
| 8-phase clause curriculum | ✅ Live | 62 questions, 70% unlock threshold |
| Wrong-book (错题集) | ✅ Live | Record / dashboard / retake one / retake all / auto-remove |
| 12 theme switcher | ✅ Live | CSS variable system, persisted in localStorage |
| Progress tracking | ✅ Live | Per-phase scores, total %, localStorage |
| Backup/restore | ✅ Live | JSON export/import with validation |
| Screen reader | ✅ Live | `aria-live` announcements on check/complete |
| Responsive (480/380/600px) | ✅ Live | Touch targets ≥44px, mobile layouts |
| Brand configurator | ✅ Live | Interactive brand picker with live preview |
| Character reference sheets | ✅ Live | Turnaround, props, expressions |
| Gramlingo branding | ✅ Live | Title, favicon, hero, nav logo |

### 2.2 Planned

| Feature | Priority | Notes |
|---------|----------|-------|
| **Preposition module** | 🔴 P0 | In/On/At — 4 phases, ~24 questions |
| Module nav tabs | 🟡 P1 | Switch between clause/preposition modules |
| Achievement system | 🟡 P1 | Badges, streaks, daily goals |
| Sound effects | 🟢 P2 | Correct/wrong/complete sounds |
| Teacher dashboard | 🟢 P2 | Class view, student progress |
| Tense module | 🟢 P3 | Past/Present/Future |
| Article module | 🟢 P3 | A/An/The |
| Mobile PWA | 🟢 P3 | Install to home screen |

---

## 3. Preposition Module Spec

### 3.1 Research Foundation

Chinese L1 interference for prepositions:
- **Omission**: Chinese uses word order, not prepositions → learners drop them entirely
- **Substitution**: 在 (zài) maps to "at/in/on" confusingly → wrong preposition chosen
- **Overuse**: Adding "in/on/at" where not needed (e.g., "discuss about")
- Source: Swan §440-450, Cambridge Learner Corpus

### 3.2 Phase Design

| Phase ID | Name | Focus | Questions |
|----------|------|-------|-----------|
| `prep_place` | In / On / At — Place | Spatial: in the box, on the table, at the station | 6-8 |
| `prep_time` | In / On / At — Time | Temporal: at 3pm, on Monday, in June | 6-8 |
| `prep_mixed` | Place + Time Mixed | Mixed contexts, choose correct preposition | 6-8 |
| `prep_prod` | Production & Editing | Fill gaps in paragraphs, correct wrong prepositions | 6-8 |

### 3.3 Sample Questions

```js
{ id: 'prep_place', name: 'In / On / At — Place', s: '空间介词', d: 'in=在里面, on=在上面, at=在具体地点', q: [
  { q: 'The cat is sleeping ___ the box.', a: 'in', o: ['in','on','at','by'], t: '在盒子里面 → in' },
  { q: 'The book is ___ the table.', a: 'on', o: ['in','on','at','to'], t: '在桌子上面 → on' },
  { q: 'I met her ___ the bus stop.', a: 'at', o: ['in','on','at','by'], t: '具体地点 → at' },
  { q: 'There is a spider ___ the ceiling.', a: 'on', o: ['in','on','at','over'], t: '天花板上 → on' },
  // ... more
] }
```

---

## 4. UX/UI Specifications

### 4.1 Color Variables (Sunset Brand)

```
--blue:     #ff8c42   Primary CTA, accent, selected borders
--bg:       #fff5ee   Page background
--black:    #2d1b10   Dark section bg
--near:     #2d1b10   Primary text
--text:     #3d2e1e   Body text
--dim:      #8a6a50   Secondary/meta text
--green:    #4caf50   Correct answer
--red:      #e07070   Wrong answer
--radius:   16px      Border radius base
--fp:       system-ui Font family
--fm:       monospace  Mono font
```

### 4.2 Component Library

| Component | Spec | CSS |
|-----------|------|-----|
| **Nav** | 48px, glass-blur, dark bg | `background: var(--nav-bg); backdrop-filter: blur(20px)` |
| **Phase Card** | 240px min, hover lift 2px | `.phase-card{background:var(--bg);border-radius:var(--radius);box-shadow:var(--card-shadow)}` |
| **Question Card** | 680px max, 12px radius | `.q-card{background:var(--bg);box-shadow:var(--qcard-shadow)}` |
| **Option Button** | 44px min, 2px border | `.opt{border:var(--opt-border);background:var(--opt-bg)}` |
| **Primary Button** | Pill shape, accent color | `.btn-primary{background:var(--blue);color:#fff}` |
| **Outline Button** | Transparent, accent border | `.btn-outline{background:transparent;border:1px solid var(--blue)}` |
| **Wrong-Bank Item** | Flex row, hover highlight | `.wrong-item{background:var(--wr-bg);border-radius:var(--radius)}` |
| **Progress Bar** | 4px, accent fill | `.progress-fill{background:var(--blue)}` |

### 4.3 Interaction States

| State | Visual |
|-------|--------|
| **Hover** | Lift 2px, shadow deepen |
| **Selected** | Accent border + tinted bg |
| **Correct** | Green bg + white text + ✓ |
| **Wrong** | Red bg + white text + ✗ |
| **Disabled** | 0.4 opacity, no pointer |
| **Focus-visible** | 3px accent outline, offset 2px |

### 4.4 Responsive Breakpoints

| Breakpoint | Changes |
|------------|---------|
| 600px | Single-column grid, safe-area padding |
| 480px | Smaller type, 40px touch targets |
| 380px | Compact cards, 36px touch targets |

---

## 5. Development Workflow

### 5.1 Pre-Development Checklist

- [ ] Read this PROJECT_BIBLE.md
- [ ] Load `english-game-builder` skill
- [ ] Verify live site matches local: `curl -sI URL | grep Content-Length` vs `wc -c index.html`
- [ ] Backup current progress with `exportProgress()`

### 5.2 Adding a Module

1. **Research** — Chinese L1 interference patterns
2. **Design** — 4 phases: recognition → guided → production → mixed
3. **Write** — 6-8 questions per phase, each with `q`, `a`, `o`, `t`
4. **Append** — Add phase objects to `DATA.phases` array
5. **Update locking** — Add new phase IDs to `isLocked()` order array
6. **Update nav** — Add module tab to `renderHome()` if multiple modules exist
7. **JS check** — `node -e "new Function(html.match(/<script>.../)[1])"` must pass
8. **Deploy** — `python deploy_game.py`
9. **Verify** — `curl | grep` for new phase IDs, browser QA (wrong → wrong-book → retake → correct removal)
10. **Update this doc** — Add new module to Feature Inventory, update counts

### 5.3 Adding a Theme

1. Copy any `[data-style="X"]` CSS block
2. Change `X` to new theme name
3. Replace all color values with new palette
4. Add `<option value="X">Name</option>` to `<select>`
5. Deploy and verify

### 5.4 Deployment

```bash
cd C:/Users/hunin/projects/relative-clause-quest
python deploy_game.py
```

The script encodes `index.html` as base64 and pushes via GitHub Contents API to `lareesawho/relative-clause-quest`. The `deploy_game.py` token must have push access to the repo.

---

## 6. File Map

| File | Purpose |
|------|---------|
| `index.html` | The entire game |
| `deploy_game.py` | Deployment script (API push) |
| `question-bank.json` | Standalone question bank (reference) |
| `PROJECT_BIBLE.md` | This document |
| `brand-configurator.html` | Interactive brand picker |
| `character-ref.html` | Full character reference page |
| `styles-20.html` | 20 style options page |
| `brand-identity.html` | Brand identity system |
| `brand.html` | Brand brainstorm |
| `v1.html` | Original brainstorm (styles 1-4) |
| `assets/gramlin-*.png` | Icon assets (5 variants) |
| `assets/gramlin-refsheet-*.png` | Character reference sheets |
| `assets/gramlingo-logo.png` | Primary wordmark logo |

---

## 7. Changelog

| Date | Change |
|------|--------|
| 2026-07-19 | PROJECT_BIBLE.md created |
| 2026-07-19 | Brand locked: Sunset Gramlin palette, Baloo 2/700/uppercase, +0.18em spacing |
| 2026-07-19 | Character reference sheets completed (turnaround, props, expressions) |
| 2026-07-19 | 12-theme style switcher deployed |
| 2026-07-18 | Gramlingo branding applied (title, favicon, hero, nav) |
| 2026-07-18 | Wrong-book (错题集) deployed |
| 2026-07-17 | Initial clause module (8 phases, 62 questions) |

---

> **Next: Preposition Module.** See §3 for spec. Follow §5.2 workflow.
> **After every dev cycle:** Update §2 (Feature Inventory) and §7 (Changelog).

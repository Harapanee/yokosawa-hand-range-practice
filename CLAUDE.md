# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ヨコサワのトーナメント用ハンドランキング（8段階の色分けシステム）を練習するプリフロップ練習ウェブアプリ。3つのシナリオ（オープン / vsオープン / vs3bet）でクイズ形式の練習ができる。

## Architecture

Single-file web app: `index.html` (HTML + CSS + JS, no build step). Open directly in a browser.

### Design System

"Velvet Table" aesthetic — luxury dark casino theme using CSS custom properties in `:root`:
- **Accent**: Champagne gold (`--gold: #c9a96e`) with light/dim variants
- **Felt**: Green table surface (`--felt` family)
- **Backgrounds**: Deep black (`--bg-deep: #08080f`)
- **Semantic colors**: `--green`/`--red` for RAISE/FOLD feedback
- **Fonts**: DM Serif Display (headings), JetBrains Mono (data/scores), DM Sans (body) — loaded from Google Fonts
- **Textures**: SVG noise overlay on body and felt via `::before` pseudo-elements

### Key Data Structures (in `<script>` block)

- **`HAND_TIERS`** — Maps each of 169 poker hands (e.g. "AKs", "77") to a tier (1-8, or 0 for fold). Tier definitions are set via `setTiers()` calls.
- **`POSITIONS`** — Array of 9 positions (UTG through BTN + SB/BB). SB/BB have `heroOnly: true` (vsオープンのヒーロー専用、オープナーにはならない).
- **`OPENER_POSITIONS`** — `POSITIONS` from UTG to BTN (heroOnly除外). オープンレイズ可能なポジション.
- **`questionLog`** — In-memory array (not persisted) storing each answered question with scenario, position, hand, tier, user/correct answers, and result.
- **Tier colors**: 1=紺, 2=赤, 3=黄, 4=緑, 5=水色, 6=白, 7=紫, 8=ピンク(BB対BTN専用). Defined in `TIER_COLORS`, `TIER_NAMES`, `TIER_TEXT`.

### Navigation

Fixed bottom tab bar with 3 tabs. Screen switching via `hideAllScreens()` + `.active` class, with corresponding `.tab-btn.active` state. Each screen gets a `screenIn` fade animation on activation.

### Table Visual

- `updateTableSeats(activePos)` dynamically generates 9 seats around an oval (open scenario)
- `updateTableSeatsVsOpen(openerName, heroName)` — opener seat in `.opener` (red), behind seats in `.behind` (grey)
- `updateTableSeatsVs3bet(heroName, threeBettorName)` — 3bettor seat in `.opener` (red), no behind seats
- Hero always at bottom center (slot 0), other seats placed clockwise via `SLOT_POSITIONS`
- Seats classified as `.hero` (gold), `.opener` (red — aggressor), `.behind` (grey — acts after hero), `.acted` (dim)
- Chip indicators (`.chip`) placed between seats and table center: `.chip-blind` (SB/BB), `.chip-raise` (opponent bets), `.chip-hero` (hero's bet)
- `TABLE_SEATS` = physical clockwise order; `ACTION_ORDER` = preflop action order
- Helper functions: `getSlotForPosition()`, `addChip()`, `addBlindsChips()`, `clearTable()`

### Quiz Logic

Three scenarios controlled by `#quiz-scenario` dropdown:

1. **オープン** (`open`) — RAISE / FOLD. `nextQuestionOpen()`. Rule: tier ≤ position.maxTier → RAISE
2. **vs オープン** (`vs-open`) — 3BET / CALL / FOLD. `nextQuestionVsOpen()`. Rules:
   - 3BET: tier ≤ opener.maxTier - 2
   - CALL: tier = opener.maxTier - 1 (BB special: CALL expanded to T6, vs BTN to T8)
   - FOLD: else
3. **vs 3bet** (`vs-3bet`) — 4BET / CALL / FOLD. `nextQuestionVs3bet()`. Rules:
   - 4BET: tier ≤ hero.maxTier - 3
   - CALL: tier = hero.maxTier - 2
   - FOLD: else. Only openable hands dealt.

- `nextQuestion()` dispatches to scenario-specific function, then calls `updateButtons()`
- `getCorrectAnswer(scenario, hand, heroPos, openerPos)` — centralized answer logic for all scenarios
- `answer(userAction)` accepts string action ('RAISE','3BET','CALL','4BET','FOLD'), uses `getCorrectAnswer()` for validation
- Score persisted via `localStorage` key `yokosawa-range-state`
- Stats display: score cards + SVG accuracy ring (circumference = 2 * PI * 23)
- Keyboard: R/F (open), 3/C/F (vs-open), 4/C/F (vs-3bet)

### UI Screens

1. **Quiz screen** (`#quiz-screen`) — Scenario selector, poker table with chip indicators, playing cards with deal animation, 2-or-3 button layout (dynamic via `updateButtons()`), accuracy ring, confetti/shake feedback
2. **Range table screen** (`#range-screen`) — 13x13 grid colored by tier. Position select uses `<optgroup>`: open-raise views + vs-open BB views. 3BET hands get orange outline, CALL hands get gold outline.
3. **Log screen** (`#log-screen`) — Scrollable answer history with filter (all/wrong only/correct only), newest first. Shows "hero vs opponent" for multi-scenario entries. Answer badges: green (RAISE/3BET/4BET), blue (CALL), red (FOLD).

## Development

```bash
open index.html   # Launch in browser (macOS)
```

No build, lint, or test commands — single HTML file with inline CSS/JS.

## Deployment

Vercel auto-deploys from `main` branch. After confirming changes work correctly (`open index.html` for local check), merge to `main` and push.

## Language

UI text is in Japanese. Code comments and variable names are in English.

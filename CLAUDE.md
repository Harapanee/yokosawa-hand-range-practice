# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ヨコサワのトーナメント用ハンドランキング（7段階の色分けシステム）を練習するプリフロップ練習ウェブアプリ。ポジションとハンドがランダムに出題され、オープンレイズすべきかフォールドすべきかを判定するクイズ形式。

## Architecture

Single-file web app: `index.html` (HTML + CSS + JS, no build step). Open directly in a browser.

### Key Data Structures (in `<script>` block)

- **`HAND_TIERS`** — Maps each of 169 poker hands (e.g. "AKs", "77") to a tier (1-7, or 0 for fold). Tier definitions are set via `setTiers()` calls.
- **`POSITIONS`** — Array of 7 positions (UTG through BTN) with `maxTier` defining which tiers can open-raise.
- **`questionLog`** — In-memory array (not persisted) storing each answered question with position, hand, tier, user/correct answers, and result.
- **Tier colors**: 1=紺, 2=赤, 3=黄, 4=緑, 5=水色, 6=白, 7=紫. Defined in `TIER_COLORS`, `TIER_NAMES`, `TIER_TEXT`.

### Table Visual

- `updateTableSeats(activePos)` dynamically generates 9 seats around an oval
- Hero always at bottom center (slot 0), other seats placed clockwise via `SLOT_POSITIONS`
- Seats classified as `.hero` (orange), `.behind` (grey — acts after hero), `.acted` (dim — already acted)
- `TABLE_SEATS` = physical clockwise order; `ACTION_ORDER` = preflop action order

### Card Rendering

- `renderPlayingCards(info)` creates two playing-card DOM elements with corners (rank+suit) and center suit
- Card colors: spade=black, heart=red, diamond=blue, club=green (`SUIT_COLORS`)

### Quiz Logic

- `nextQuestion()` picks random position (or filtered) + weighted hand selection (50% in-range / 50% out-of-range), updates table visual and card display
- `answer(isRaise)` checks if `handTier >= 1 && handTier <= position.maxTier`, logs result, shows last-result bar with fade-out animation, screen flashes green/red, then immediately advances to next question
- Score persisted via `localStorage` key `yokosawa-range-state`
- Position filter (`#quiz-position-filter`): practice all positions or a specific one

### UI Screens

Three screens toggled via `.active` class (managed by `hideAllScreens()` + add `.active`):
1. **Quiz screen** (`#quiz-screen`) — Poker table visual (self-perspective, seats rotate), playing cards, RAISE/FOLD buttons, last-result fade bar, position filter
2. **Range table screen** (`#range-screen`) — 13x13 grid colored by tier, position filter with in-range highlight (yellow outline) and out-of-range dimming
3. **Log screen** (`#log-screen`) — Scrollable answer history with filter (all/wrong only/correct only), newest first

### Visual Feedback

- Screen flash overlay via `#quiz-screen::before` pseudo-element with CSS animation (`flash-correct`/`flash-wrong`)
- Last result bar (`#last-result`) shows previous answer's result with `resultFadeLife` animation (1.5s slide-in + fade-out)

## Development

```bash
open index.html   # Launch in browser (macOS)
```

No build, lint, or test commands — single HTML file with inline CSS/JS.

## Language

UI text is in Japanese. Code comments and variable names are in English.

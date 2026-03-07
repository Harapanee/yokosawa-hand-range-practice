# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ヨコサワのトーナメント用ハンドランキング（8段階の色分けシステム）を練習するプリフロップ練習ウェブアプリ。3つのシナリオ（オープン / vsオープン / vs3bet）でクイズ形式の練習ができる。

## Architecture

Single-file web app: `index.html` (~2500 lines of HTML + CSS + JS, no build step). Open directly in a browser. CSS (lines ~9–1140), HTML (lines ~1040–1143 interleaved), JS (lines ~1145–2497). `docs/plans/` contains archived design documents. `docs/rules.md` documents the complete answer-checking rules with examples.

### Design System

"Velvet Table" aesthetic — luxury dark casino theme using CSS custom properties in `:root`:
- **Accent**: Champagne gold (`--gold: #c9a96e`) with light/dim variants
- **Felt**: Green table surface (`--felt` family)
- **Backgrounds**: Deep black (`--bg-deep: #08080f`)
- **Semantic colors**: `--green`/`--red` for correct/incorrect feedback
- **Fonts**: DM Serif Display (headings), JetBrains Mono (data/scores), DM Sans (body) — loaded from Google Fonts

### Key Data Structures (in `<script>` block)

- **`HAND_TIERS`** — Maps each of 169 poker hands (e.g. "AKs", "77") to a tier (1-8, or 0 for fold). Tier definitions are set via `setTiers()` calls.
- **`POSITIONS`** — Array of 9 positions (UTG through BTN + SB/BB). Each has `maxTier` defining their opening range width. Only BB has `heroOnly: true` (excluded from `OPENER_POSITIONS`). SB opens with same range as BTN (maxTier 7).
- **`OPENER_POSITIONS`** — `POSITIONS` filtered to exclude `heroOnly`. オープンレイズ可能なポジション.
- **`questionLog`** — In-memory array (not persisted) storing each answered question. Each entry has `opponentPosition` (opponent name, or null for open scenario).
- **`state`** — Main app state: score, current hand/position, scenario, chain status, 3bettor info.
- **`animationController`** — `{ abortController, isAnimating }` for managing async table animations.
- **Tier colors**: 1=紺, 2=赤, 3=黄, 4=緑, 5=水色, 6=白, 7=紫, 8=ピンク(BB対BTN専用). Defined in `TIER_COLORS`, `TIER_NAMES`, `TIER_TEXT`.

### Table Visual & Animation System

- `setupTableSeatsWaiting(heroName, behindSet)` creates initial table with seats in `.waiting` or `.behind` state
- `animateFoldSequence(config, signal)` — async animated sequence: seats fold one-by-one until hero/opener/3bettor are revealed
- `playPostAction(correctAction, signal)` — after correct answer, shows hero's chip and folds behind seats
- `tryChainTo3bet(signal)` — 40% chance after open RAISE: a behind player 3bets, transitioning to vs-3bet scenario
- Seat transitions: `transitionToFold()`, `transitionToHero()`, `transitionToOpener()`, `transitionToBehind()`
- All animations are abortable via `AbortController` / `signal` pattern using `sleep(ms, signal)`
- Hero always at bottom center (slot 0), other seats placed clockwise via `SLOT_POSITIONS`
- `TABLE_SEATS` = physical clockwise order; `ACTION_ORDER` = preflop action order

### Quiz Logic

Three scenarios controlled by `#quiz-scenario` dropdown:

1. **オープン** (`open`) — RAISE / FOLD. Rule: `tier ≤ position.maxTier → RAISE`
2. **vs オープン** (`vs-open`) — 3BET / CALL / FOLD. Rules:
   - 3BET: `tier ≤ openerMax - 2`
   - CALL: `tier = openerMax - 1` (BB special: CALL expanded to T6, vs CO to T7, vs BTN to T8)
   - FOLD: else
3. **vs 3bet** (`vs-3bet`) — 4BET / CALL / FOLD. Rules:
   - 4BET: `tier === 1` (always) or `tier ≤ heroMax - 4`
   - CALL: `tier ≤ heroMax - 2`
   - FOLD: else. Only openable hands dealt.

- `nextQuestion()` is async — sets scenario, calls `updateButtons()`, then dispatches to scenario-specific async function with fold animation
- Buttons are shown BEFORE animation completes — users can answer during the fold sequence animation
- `answer(userAction)` is async — guarded by `state.answered` only (not `isAnimating`). On correct: runs post-action animation + optional chain. On vs-3bet: reveals opponent's hand with `showOpponentHand()`.
- `getCorrectAnswer(scenario, hand, heroPos, openerPos)` — centralized answer logic
- Score persisted via `localStorage` key `yokosawa-range-state`
- Keyboard: R/F (open), 3/C/F (vs-open), 4/C/F (vs-3bet)
- Visual feedback: ○ (green) for correct, ✕ (red) for incorrect — `showResultSymbol()`

### Chain System (open → vs-3bet)

When user correctly RAISEs in open scenario, `tryChainTo3bet()` has 40% chance to trigger:
1. Hero's 3bb chip appears
2. Behind seats process: one random seat becomes 3bettor (`.opener` + 9bb chip), others fold
3. State transitions to `vs-3bet` with `state.isChained = true`
4. Buttons update to 4BET/CALL/FOLD, user answers the chained question
5. After chained answer, `nextQuestion()` resumes normal flow

### UI Screens

1. **Quiz screen** (`#quiz-screen`) — Scenario selector, poker table with animated fold sequence, playing cards with deal animation, action buttons, result symbol feedback
2. **Range table screen** (`#range-screen`) — 13x13 grid colored by tier. Position select uses `<optgroup>`: open-raise views + vs-open BB views. 3BET hands get orange outline, CALL hands get gold outline.
3. **Log screen** (`#log-screen`) — Scrollable answer history with filter (all/wrong only/correct only), newest first. Each entry shows scenario badge, position, hand, user answer, and correct answer.

Quiz screen has a position filter dropdown (`populateQuizPositionFilter`) to restrict which positions appear (open scenario only; hidden for vs-open/vs-3bet).

### Navigation

Bottom tab bar with auto-hide on scroll down, show on scroll up.

## Development

```bash
open index.html   # Launch in browser (macOS)
```

No build, lint, or test commands — single HTML file with inline CSS/JS.

## Deployment

Vercel auto-deploys from `main` branch. After confirming changes work correctly (`open index.html` for local check), merge to `main` and push.

## Language

UI text is in Japanese. Code comments and variable names are in English.

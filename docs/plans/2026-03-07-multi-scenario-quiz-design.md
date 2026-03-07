# Multi-Scenario Quiz Design

## Overview

Add two new quiz scenarios (vs Open, vs 3bet) to the existing open-raise quiz, supporting Yokosawa's full preflop decision system.

## Scenarios

### 1. Open Raise (existing)
- Buttons: RAISE / FOLD
- Rule: tier <= position.maxTier -> RAISE

### 2. vs Open (new)
- Buttons: 3BET / CALL / FOLD
- Rule:
  - 3BET: tier <= opener.maxTier - 2
  - CALL: tier = opener.maxTier - 1
  - FOLD: tier >= opener.maxTier or tier = 0
- BB special rule:
  - vs non-BTN: CALL expanded to tier <= 6
  - vs BTN: CALL expanded to tier <= 8 (pink)
  - 3BET: standard rule applies

### 3. vs 3bet (new)
- Buttons: 4BET / CALL / FOLD
- Rule:
  - 4BET: tier <= hero.maxTier - 3
  - CALL: tier = hero.maxTier - 2
  - FOLD: tier >= hero.maxTier - 1 or tier = 0
- Only deals hands from hero's opening range (tier <= hero.maxTier)

## Data Changes

### Pink Tier (T8) - BB vs BTN only
Suited: J5s, J4s, J3s, J2s, T6s, T5s, T4s, T3s, 95s, 85s, 74s, 63s, 53s, 43s
Offsuit: K8o, Q8o, J8o, T8o, K7o, Q7o, 97o, 87o, A5o, A4o, A3o, A2o, K6o, K5o

### New positions for hero (vs Open only)
- SB and BB added as possible hero positions
- SB/BB are NOT openers

## Question Generation

- **vs Open**: random opener -> random hero behind opener -> random hand
- **vs 3bet**: random hero opening position -> hand from opening range -> random 3bettor behind hero

## UI Changes

- Scenario dropdown above position filter: "オープン / vs オープン / vs 3bet"
- Button row: 3 buttons when in vs-open or vs-3bet mode
- Position label format:
  - Open: `BTN（後ろに2人）`
  - vs Open: `あなた: CO <- UTGのオープン`
  - vs 3bet: `あなた: BTNオープン -> COの3bet`
- Table visual: opener/3bettor seat highlighted in red
- Keyboard: R=RAISE, C=CALL, 3=3BET, 4=4BET, F=FOLD
- Range table: add vs-open and vs-3bet view options

## Edge Cases

- UTG open -> 4BET requires tier <= 0: impossible, only CALL(T1) or FOLD
- vs 3bet only deals openable hands, ensuring CALL/4BET answers exist
- SB not used as opener position

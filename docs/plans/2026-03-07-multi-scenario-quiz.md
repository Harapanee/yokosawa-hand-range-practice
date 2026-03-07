# Multi-Scenario Quiz Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add vs-open (3BET/CALL/FOLD) and vs-3bet (4BET/CALL/FOLD) quiz scenarios to the existing open-raise quiz.

**Architecture:** Single-file app (`index.html`). Add pink tier data, scenario selection dropdown, 3-button layout, updated question generation logic, and table visual updates. All changes in one file.

**Tech Stack:** Vanilla HTML/CSS/JS, no build step.

**Design doc:** `docs/plans/2026-03-07-multi-scenario-quiz-design.md`

---

### Task 1: Add Pink Tier Data + SB/BB Positions

**Files:**
- Modify: `index.html:960-1015` (DATA section)

**Step 1: Add T8 to tier color/name/text maps**

In `TIER_COLORS`, `TIER_NAMES`, `TIER_TEXT`, add tier 8:

```js
// TIER_COLORS — add:
8: '#e91e63'

// TIER_NAMES — add:
8: 'ピンク - BB対BTN'

// TIER_TEXT — add:
8: '#fff'
```

**Step 2: Add pink tier hand data**

After the existing `setTiers(7, ...)` line (~line 1005), add:

```js
setTiers(8, [
  'J5s','J4s','J3s','J2s','T6s','T5s','T4s','T3s','95s','85s','74s','63s','53s','43s',
  'K8o','Q8o','J8o','T8o','K7o','Q7o','97o','87o','A5o','A4o','A3o','A2o','K6o','K5o'
]);
```

**Step 3: Add SB and BB to position data**

After the BTN entry in `POSITIONS` array, add SB and BB. These are used only as hero positions in vs-open scenarios:

```js
{ name: 'SB', players: 1, maxTier: 0, heroOnly: true },
{ name: 'BB', players: 0, maxTier: 0, heroOnly: true },
```

`maxTier: 0` because SB/BB don't open-raise in this system. `heroOnly: true` marks them as non-opener positions.

**Step 4: Add SB/BB to TABLE_SEATS if not already present**

`TABLE_SEATS` already includes SB and BB: `['BTN','SB','BB','UTG','UTG+1','MP','LJ','HJ','CO']`. No change needed.

**Step 5: Verify in browser**

Open `index.html`. Existing quiz should work unchanged. Check range table — T8 hands should not appear (they're fold-colored since no position has maxTier >= 8 for open).

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add pink tier (T8) data and SB/BB positions"
```

---

### Task 2: Add Scenario Selection UI

**Files:**
- Modify: `index.html:878-883` (quiz filter HTML)
- Modify: `index.html:456-515` (button CSS)

**Step 1: Add scenario dropdown HTML**

Replace the quiz-filter div (~line 878-883) with:

```html
<div class="quiz-filter">
  <span>シナリオ:</span>
  <select id="quiz-scenario" onchange="onScenarioChange()">
    <option value="open">オープン</option>
    <option value="vs-open">vs オープン</option>
    <option value="vs-3bet">vs 3bet</option>
  </select>
  <span style="margin-left:8px;">ポジション:</span>
  <select id="quiz-position-filter" onchange="nextQuestion()">
    <option value="all">全ポジション</option>
  </select>
</div>
```

**Step 2: Add CSS for 3-button layout**

Add after the existing `.btn-fold` styles (~line 515):

```css
.btn-3bet {
  background: linear-gradient(180deg, #ff9800, #e65100);
  color: #fff;
  box-shadow:
    0 4px 16px rgba(230,81,0,0.35),
    inset 0 -2px 0 rgba(0,0,0,0.15),
    inset 0 1px 0 rgba(255,255,255,0.15);
  text-shadow: 0 1px 2px rgba(0,0,0,0.2);
}
.btn-call {
  background: linear-gradient(180deg, #42a5f5, #1565c0);
  color: #fff;
  box-shadow:
    0 4px 16px rgba(21,101,192,0.35),
    inset 0 -2px 0 rgba(0,0,0,0.15),
    inset 0 1px 0 rgba(255,255,255,0.15);
  text-shadow: 0 1px 2px rgba(0,0,0,0.2);
}
.btn-3bet.chosen {
  box-shadow: 0 0 0 3px rgba(255,152,0,0.6), 0 0 24px rgba(255,152,0,0.3);
}
.btn-call.chosen {
  box-shadow: 0 0 0 3px rgba(66,165,245,0.6), 0 0 24px rgba(66,165,245,0.3);
}
```

**Step 3: Verify in browser**

Scenario dropdown appears. Buttons still show RAISE/FOLD. Selecting scenarios doesn't do anything yet.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add scenario dropdown and 3-button CSS"
```

---

### Task 3: Core Quiz Logic for vs-Open

**Files:**
- Modify: `index.html` JS section — `nextQuestion()`, `answer()`, and new helper functions

**Step 1: Add scenario state and correct-answer calculator**

Add after the `state` object definition (~line 1087):

```js
function getCorrectAnswer(scenario, hand, heroPos, openerPos) {
  const tier = getTier(hand);
  if (tier === 0) return 'FOLD';

  if (scenario === 'open') {
    return tier <= heroPos.maxTier ? 'RAISE' : 'FOLD';
  }

  if (scenario === 'vs-open') {
    const openerMax = openerPos.maxTier;
    // BB special rule
    if (heroPos.name === 'BB') {
      const threeBetThresh = openerMax - 2;
      if (tier <= threeBetThresh) return '3BET';
      const callMax = openerPos.name === 'BTN' ? 8 : 6;
      if (tier <= callMax) return 'CALL';
      return 'FOLD';
    }
    // SB rule — same as standard non-BB
    if (tier <= openerMax - 2) return '3BET';
    if (tier === openerMax - 1) return 'CALL';
    return 'FOLD';
  }

  if (scenario === 'vs-3bet') {
    const heroMax = heroPos.maxTier;
    if (tier <= heroMax - 3) return '4BET';
    if (tier === heroMax - 2) return 'CALL';
    return 'FOLD';
  }

  return 'FOLD';
}
```

**Step 2: Add opener position for action order lookup**

Add helper to get valid positions for each scenario:

```js
const OPENER_POSITIONS = POSITIONS.filter(p => !p.heroOnly);

function getHeroPositionsBehind(openerName) {
  const order = ACTION_ORDER;
  const opIdx = order.indexOf(openerName);
  const behind = order.slice(opIdx + 1);
  // Map to position objects, add SB/BB
  return behind.map(name => {
    return POSITIONS.find(p => p.name === name) ||
           { name, players: 0, maxTier: 0 };
  });
}
```

**Step 3: Update `nextQuestion()` for vs-open scenario**

Rewrite `nextQuestion()` to handle all scenarios:

```js
function nextQuestion() {
  state.answered = false;
  const scenario = document.getElementById('quiz-scenario').value;
  state.scenario = scenario;

  if (scenario === 'open') {
    nextQuestionOpen();
  } else if (scenario === 'vs-open') {
    nextQuestionVsOpen();
  } else if (scenario === 'vs-3bet') {
    nextQuestionVs3bet();
  }

  updateButtons();
}

function nextQuestionOpen() {
  const posFilter = document.getElementById('quiz-position-filter').value;
  if (posFilter === 'all') {
    state.currentPosition = OPENER_POSITIONS[Math.floor(Math.random() * OPENER_POSITIONS.length)];
  } else {
    state.currentPosition = POSITIONS[parseInt(posFilter)];
  }
  state.openerPosition = null;

  const pos = state.currentPosition;
  const inRange = ALL_HANDS.filter(h => { const t = getTier(h); return t >= 1 && t <= pos.maxTier; });
  const outRange = ALL_HANDS.filter(h => { const t = getTier(h); return t === 0 || t > pos.maxTier; });

  if (Math.random() < 0.5 && inRange.length > 0) {
    state.currentHand = inRange[Math.floor(Math.random() * inRange.length)];
  } else if (outRange.length > 0) {
    state.currentHand = outRange[Math.floor(Math.random() * outRange.length)];
  } else {
    state.currentHand = ALL_HANDS[Math.floor(Math.random() * ALL_HANDS.length)];
  }

  document.getElementById('position-label').textContent = `${pos.name}（後ろに${pos.players}人）`;
  updateTableSeats(pos.name);
  renderHand();
}

function nextQuestionVsOpen() {
  // Pick random opener
  const opener = OPENER_POSITIONS[Math.floor(Math.random() * OPENER_POSITIONS.length)];
  state.openerPosition = opener;

  // Pick random hero behind opener
  const heroCandidates = getHeroPositionsBehind(opener.name);
  const hero = heroCandidates[Math.floor(Math.random() * heroCandidates.length)];
  state.currentPosition = hero;

  // Pick hand — 50/50 split between actionable and fold
  const allAnswers = ALL_HANDS.map(h => ({
    hand: h,
    answer: getCorrectAnswer('vs-open', h, hero, opener)
  }));
  const actionable = allAnswers.filter(a => a.answer !== 'FOLD');
  const foldable = allAnswers.filter(a => a.answer === 'FOLD');

  if (Math.random() < 0.5 && actionable.length > 0) {
    const pick = actionable[Math.floor(Math.random() * actionable.length)];
    state.currentHand = pick.hand;
  } else if (foldable.length > 0) {
    const pick = foldable[Math.floor(Math.random() * foldable.length)];
    state.currentHand = pick.hand;
  } else {
    state.currentHand = ALL_HANDS[Math.floor(Math.random() * ALL_HANDS.length)];
  }

  document.getElementById('position-label').innerHTML =
    `あなた: <strong>${hero.name}</strong> &larr; ${opener.name}のオープン`;
  updateTableSeatsVsOpen(opener.name, hero.name);
  renderHand();
}

function renderHand() {
  const info = handDisplayInfo(state.currentHand);
  document.getElementById('hand-name').textContent = info.name;
  renderPlayingCards(info);
}
```

**Step 4: Add `updateTableSeatsVsOpen()` for table visual**

```js
function updateTableSeatsVsOpen(openerName, heroName) {
  const table = document.getElementById('poker-table');
  table.querySelectorAll('.seat').forEach(el => el.remove());

  const heroIdx = TABLE_SEATS.indexOf(heroName);

  for (let s = 0; s < 9; s++) {
    const seatIdx = (heroIdx + s) % 9;
    const posName = TABLE_SEATS[seatIdx];
    const slot = SLOT_POSITIONS[s];

    const el = document.createElement('div');
    el.className = 'seat';
    el.style.top = slot.top;
    el.style.left = slot.left;

    const label = posName === 'UTG+1' ? 'UTG1' : posName;
    if (s === 0) {
      el.classList.add('hero');
      el.innerHTML = label + '<span class="hero-label">YOU</span>';
    } else if (posName === openerName) {
      el.classList.add('behind'); // reuse style, will add opener style in Task 5
      el.style.background = 'rgba(168,50,50,0.75)';
      el.style.color = '#ffaaaa';
      el.style.borderColor = 'rgba(208,64,64,0.5)';
      el.textContent = label;
    } else {
      el.classList.add('acted');
      el.textContent = label;
    }

    table.appendChild(el);
  }
}
```

**Step 5: Verify in browser**

Select "vs オープン" scenario. Should show opener and hero positions, deal cards. Buttons still show RAISE/FOLD (updated in next task).

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add vs-open question generation and table visual"
```

---

### Task 4: Update Buttons and Answer Logic

**Files:**
- Modify: `index.html` — button HTML, `updateButtons()`, `answer()`, keyboard handler

**Step 1: Update button HTML to support dynamic labels**

Replace the btn-row div (~line 895-898):

```html
<div class="btn-row" id="btn-row">
  <button class="btn btn-raise" id="btn-raise" onclick="answer('RAISE')">RAISE</button>
  <button class="btn btn-fold" id="btn-fold" onclick="answer('FOLD')">FOLD</button>
</div>
```

**Step 2: Add `updateButtons()` function**

```js
function updateButtons() {
  const row = document.getElementById('btn-row');
  const scenario = state.scenario || 'open';

  if (scenario === 'open') {
    row.innerHTML = `
      <button class="btn btn-raise" onclick="answer('RAISE')">RAISE</button>
      <button class="btn btn-fold" onclick="answer('FOLD')">FOLD</button>`;
  } else if (scenario === 'vs-open') {
    row.innerHTML = `
      <button class="btn btn-3bet" onclick="answer('3BET')">3BET</button>
      <button class="btn btn-call" onclick="answer('CALL')">CALL</button>
      <button class="btn btn-fold" onclick="answer('FOLD')">FOLD</button>`;
  } else if (scenario === 'vs-3bet') {
    row.innerHTML = `
      <button class="btn btn-3bet" onclick="answer('4BET')">4BET</button>
      <button class="btn btn-call" onclick="answer('CALL')">CALL</button>
      <button class="btn btn-fold" onclick="answer('FOLD')">FOLD</button>`;
  }
}
```

**Step 3: Rewrite `answer()` to use string-based actions**

```js
function answer(userAction) {
  if (state.answered) return;
  state.answered = true;

  const scenario = state.scenario || 'open';
  const hand = state.currentHand;
  const tier = getTier(hand);
  const correctAction = getCorrectAnswer(scenario, hand, state.currentPosition, state.openerPosition);
  const isCorrect = userAction === correctAction;

  state.total++;
  if (isCorrect) {
    state.correct++;
    state.streak++;
    if (state.streak > state.bestStreak) state.bestStreak = state.streak;
  } else {
    state.streak = 0;
  }

  updateStats();
  saveState();

  questionLog.unshift({
    scenario: scenario,
    position: state.currentPosition.name,
    openerPosition: state.openerPosition ? state.openerPosition.name : null,
    hand: hand,
    tier: tier,
    userAnswer: userAction,
    correctAnswer: correctAction,
    isCorrect: isCorrect,
  });

  // Last result bar
  const tierColor = TIER_COLORS[tier] || TIER_COLORS[0];
  const tierName = tier === 0 ? 'フォールド' : `T${tier}`;
  const lastResult = document.getElementById('last-result');
  const cls = isCorrect ? 'correct' : 'wrong';
  const icon = isCorrect ? '&#10004;' : '&#10008;';
  const posInfo = state.openerPosition
    ? `${state.currentPosition.name} vs ${state.openerPosition.name}`
    : state.currentPosition.name;
  lastResult.innerHTML = `<div class="last-result-inner ${cls}">
    <span class="lr-icon">${icon}</span>
    <span>${posInfo}</span>
    <span>${hand}</span>
    <span class="lr-tier-badge" style="background:${tierColor}"></span>
    <span class="lr-detail">${tierName} → ${correctAction}</span>
  </div>`;

  // Screen flash
  const screen = document.getElementById('quiz-screen');
  screen.classList.remove('flash-correct', 'flash-wrong');
  void screen.offsetWidth;
  screen.classList.add(isCorrect ? 'flash-correct' : 'flash-wrong');

  if (isCorrect) {
    spawnConfetti();
  } else {
    const btnRow = document.getElementById('btn-row');
    btnRow.classList.remove('shake');
    void btnRow.offsetWidth;
    btnRow.classList.add('shake');
  }

  nextQuestion();
}
```

**Step 4: Update keyboard handler**

Replace the keydown listener:

```js
document.addEventListener('keydown', (e) => {
  if (!document.getElementById('quiz-screen').classList.contains('active')) return;
  const scenario = state.scenario || 'open';
  const key = e.key.toUpperCase();

  if (scenario === 'open') {
    if (key === 'R') answer('RAISE');
    if (key === 'F') answer('FOLD');
  } else if (scenario === 'vs-open') {
    if (key === '3') answer('3BET');
    if (key === 'C') answer('CALL');
    if (key === 'F') answer('FOLD');
  } else if (scenario === 'vs-3bet') {
    if (key === '4') answer('4BET');
    if (key === 'C') answer('CALL');
    if (key === 'F') answer('FOLD');
  }
});
```

**Step 5: Add `onScenarioChange()` and update keyboard hint**

```js
function onScenarioChange() {
  const scenario = document.getElementById('quiz-scenario').value;
  const hint = document.querySelector('.kb-hint');
  if (scenario === 'open') {
    hint.textContent = 'R / F キー対応';
  } else if (scenario === 'vs-open') {
    hint.textContent = '3 / C / F キー対応';
  } else if (scenario === 'vs-3bet') {
    hint.textContent = '4 / C / F キー対応';
  }
  nextQuestion();
}
```

**Step 6: Verify in browser**

- Select "オープン" — RAISE/FOLD buttons, existing behavior
- Select "vs オープン" — 3BET/CALL/FOLD buttons, shows opener vs hero
- Answer questions, check correctness

**Step 7: Commit**

```bash
git add index.html
git commit -m "feat: dynamic buttons and answer logic for all scenarios"
```

---

### Task 5: vs 3bet Scenario

**Files:**
- Modify: `index.html` JS section

**Step 1: Add `nextQuestionVs3bet()`**

```js
function nextQuestionVs3bet() {
  // Pick random hero opening position
  const hero = OPENER_POSITIONS[Math.floor(Math.random() * OPENER_POSITIONS.length)];
  state.currentPosition = hero;

  // Pick random 3bettor behind hero
  const threeBettorCandidates = getHeroPositionsBehind(hero.name);
  const threeBettor = threeBettorCandidates[Math.floor(Math.random() * threeBettorCandidates.length)];
  state.openerPosition = hero; // hero is the original opener
  state.threeBettorPosition = threeBettor;

  // Only deal hands hero would have opened
  const openableHands = ALL_HANDS.filter(h => {
    const t = getTier(h);
    return t >= 1 && t <= hero.maxTier;
  });

  // 50/50 between continue and fold
  const allAnswers = openableHands.map(h => ({
    hand: h,
    answer: getCorrectAnswer('vs-3bet', h, hero, null)
  }));
  const continuable = allAnswers.filter(a => a.answer !== 'FOLD');
  const foldable = allAnswers.filter(a => a.answer === 'FOLD');

  if (Math.random() < 0.5 && continuable.length > 0) {
    const pick = continuable[Math.floor(Math.random() * continuable.length)];
    state.currentHand = pick.hand;
  } else if (foldable.length > 0) {
    const pick = foldable[Math.floor(Math.random() * foldable.length)];
    state.currentHand = pick.hand;
  } else {
    state.currentHand = openableHands[Math.floor(Math.random() * openableHands.length)];
  }

  document.getElementById('position-label').innerHTML =
    `あなた: <strong>${hero.name}</strong>オープン &rarr; ${threeBettor.name}の3bet`;
  updateTableSeatsVs3bet(hero.name, threeBettor.name);
  renderHand();
}
```

**Step 2: Add `updateTableSeatsVs3bet()`**

```js
function updateTableSeatsVs3bet(heroName, threeBettorName) {
  const table = document.getElementById('poker-table');
  table.querySelectorAll('.seat').forEach(el => el.remove());

  const heroIdx = TABLE_SEATS.indexOf(heroName);

  for (let s = 0; s < 9; s++) {
    const seatIdx = (heroIdx + s) % 9;
    const posName = TABLE_SEATS[seatIdx];
    const slot = SLOT_POSITIONS[s];

    const el = document.createElement('div');
    el.className = 'seat';
    el.style.top = slot.top;
    el.style.left = slot.left;

    const label = posName === 'UTG+1' ? 'UTG1' : posName;
    if (s === 0) {
      el.classList.add('hero');
      el.innerHTML = label + '<span class="hero-label">YOU</span>';
    } else if (posName === threeBettorName) {
      el.classList.add('behind');
      el.style.background = 'rgba(168,50,50,0.75)';
      el.style.color = '#ffaaaa';
      el.style.borderColor = 'rgba(208,64,64,0.5)';
      el.textContent = label;
    } else {
      el.classList.add('acted');
      el.textContent = label;
    }

    table.appendChild(el);
  }
}
```

**Step 3: Update `answer()` for vs-3bet position info**

In the `answer()` function, update the `posInfo` variable:

```js
const posInfo = scenario === 'vs-3bet'
  ? `${state.currentPosition.name} vs ${state.threeBettorPosition.name}`
  : state.openerPosition
    ? `${state.currentPosition.name} vs ${state.openerPosition.name}`
    : state.currentPosition.name;
```

**Step 4: Verify in browser**

- Select "vs 3bet", answer questions
- Check that only openable hands are dealt
- Check 4BET/CALL/FOLD correctness for various positions

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add vs-3bet quiz scenario"
```

---

### Task 6: Update Range Table for New Scenarios

**Files:**
- Modify: `index.html` — `renderRange()`, range position select

**Step 1: Update range position select options**

Rewrite `populatePositionSelect()`:

```js
function populatePositionSelect() {
  const sel = document.getElementById('range-position');
  // Open raise options
  const openGroup = document.createElement('optgroup');
  openGroup.label = 'オープンレイズ';
  OPENER_POSITIONS.forEach((p, i) => {
    const opt = document.createElement('option');
    opt.value = `open-${i}`;
    opt.textContent = `${p.name} (${p.players}人) - T1~T${p.maxTier}`;
    openGroup.appendChild(opt);
  });
  sel.appendChild(openGroup);

  // vs Open options
  const vsOpenGroup = document.createElement('optgroup');
  vsOpenGroup.label = 'vs オープン (BB)';
  OPENER_POSITIONS.forEach((p, i) => {
    const opt = document.createElement('option');
    opt.value = `vs-open-bb-${i}`;
    opt.textContent = `BB vs ${p.name}オープン`;
    vsOpenGroup.appendChild(opt);
  });
  sel.appendChild(vsOpenGroup);
}
```

**Step 2: Update `renderRange()` for vs-open view**

```js
function renderRange() {
  const grid = document.getElementById('range-grid');
  grid.innerHTML = '';

  const selVal = document.getElementById('range-position').value;

  let mode = 'all';
  let pos = null;
  let openerPos = null;
  let heroPos = null;

  if (selVal === '-1') {
    mode = 'all';
  } else if (selVal.startsWith('open-')) {
    mode = 'open';
    pos = OPENER_POSITIONS[parseInt(selVal.split('-')[1])];
  } else if (selVal.startsWith('vs-open-bb-')) {
    mode = 'vs-open';
    const idx = parseInt(selVal.split('-').pop());
    openerPos = OPENER_POSITIONS[idx];
    heroPos = POSITIONS.find(p => p.name === 'BB');
  }

  for (let r = 0; r < 13; r++) {
    for (let c = 0; c < 13; c++) {
      const hand = gridToHand(r, c);
      const tier = getTier(hand);
      const cell = document.createElement('div');
      cell.className = 'range-cell';
      cell.textContent = hand;

      if (tier === 0) {
        cell.classList.add('fold-cell');
      } else {
        cell.style.background = TIER_COLORS[tier];
        cell.style.color = TIER_TEXT[tier];
      }

      if (mode === 'open' && pos) {
        if (tier >= 1 && tier <= pos.maxTier) {
          cell.classList.add('in-range');
        } else {
          cell.classList.add('out-of-range');
        }
      } else if (mode === 'vs-open' && openerPos && heroPos) {
        const action = getCorrectAnswer('vs-open', hand, heroPos, openerPos);
        if (action === '3BET') {
          cell.classList.add('in-range');
          cell.style.outline = '2px solid #ff9800';
        } else if (action === 'CALL') {
          cell.classList.add('in-range');
        } else {
          cell.classList.add('out-of-range');
        }
      }

      grid.appendChild(cell);
    }
  }
}
```

**Step 3: Update legend for pink tier**

In `buildLegend()`, change the loop to include T8:

```js
for (let t = 1; t <= 8; t++) {
```

**Step 4: Verify in browser**

- Range table: "全ティア表示" shows all 8 tiers including pink
- Open raise options work as before
- "BB vs BTNオープン" shows 3BET (orange outline) and CALL (gold outline) ranges

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: update range table for vs-open views"
```

---

### Task 7: Update Log Screen for Scenarios

**Files:**
- Modify: `index.html` — `renderLog()`

**Step 1: Update log entry rendering**

In `renderLog()`, update the entry template to show scenario info:

```js
list.innerHTML = entries.map(e => {
  const tierColor = TIER_COLORS[e.tier] || TIER_COLORS[0];
  const tierLabel = e.tier === 0 ? 'Fold' : `T${e.tier}`;
  const iconClass = e.isCorrect ? 'correct' : 'wrong';
  const icon = e.isCorrect ? '&#10004;' : '&#10008;';
  const ansClass = e.userAnswer === 'FOLD' ? 'fold' : 'raise';
  const posText = e.openerPosition
    ? `${e.position} vs ${e.openerPosition}`
    : e.position;
  return `<div class="log-entry">
    <span class="log-icon ${iconClass}">${icon}</span>
    <span class="log-pos">${posText}</span>
    <span class="log-hand">${e.hand}</span>
    <span class="log-answer ${ansClass}">${e.userAnswer}</span>
    <span class="log-detail"><span class="log-tier-badge" style="background:${tierColor}"></span>${tierLabel} → ${e.correctAnswer}</span>
  </div>`;
}).join('');
```

**Step 2: Verify in browser**

Answer some questions in each scenario. Log shows position info like "CO vs UTG" for vs-open.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: update log screen for multi-scenario entries"
```

---

### Task 8: Polish and Edge Cases

**Files:**
- Modify: `index.html`

**Step 1: Add CSS for opener seat highlight**

```css
.poker-table .seat.opener {
  background: rgba(168,50,50,0.75);
  color: #ffaaaa;
  border-color: rgba(208,64,64,0.5);
  box-shadow: 0 0 12px rgba(208,64,64,0.3);
}
```

Replace inline styles in `updateTableSeatsVsOpen()` and `updateTableSeatsVs3bet()` with `el.classList.add('opener')`.

**Step 2: Handle edge case — UTG vs 3bet has no 4BET**

In `nextQuestionVs3bet()`, if hero.maxTier <= 3, there are no 4BET-eligible hands (maxTier-3 <= 0). The quiz still works because CALL and FOLD are valid answers. No special handling needed.

**Step 3: Update position filter for each scenario**

Add `onScenarioChange()` logic to repopulate position filter:

```js
function onScenarioChange() {
  const scenario = document.getElementById('quiz-scenario').value;
  const sel = document.getElementById('quiz-position-filter');
  sel.innerHTML = '<option value="all">全ポジション</option>';

  if (scenario === 'open') {
    OPENER_POSITIONS.forEach((p, i) => {
      const opt = document.createElement('option');
      opt.value = i;
      opt.textContent = p.name;
      sel.appendChild(opt);
    });
  }
  // vs-open and vs-3bet: position filter not used (both positions are randomized)

  const hint = document.querySelector('.kb-hint');
  if (scenario === 'open') {
    hint.textContent = 'R / F キー対応';
  } else if (scenario === 'vs-open') {
    hint.textContent = '3 / C / F キー対応';
  } else if (scenario === 'vs-3bet') {
    hint.textContent = '4 / C / F キー対応';
  }

  nextQuestion();
}
```

**Step 4: Full browser verification**

Test all scenarios:
1. Open — RAISE/FOLD works as before
2. vs Open — 3BET/CALL/FOLD with BB special rules
3. vs 3bet — 4BET/CALL/FOLD, only openable hands
4. Range table — all views
5. Log — shows scenario info
6. Keyboard shortcuts work
7. Score persists via localStorage

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: polish multi-scenario quiz — opener styling, position filter, edge cases"
```

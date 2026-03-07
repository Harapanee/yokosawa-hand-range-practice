# Duolingo風ポップUI リデザイン Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** index.html のCSS・HTMLを全面的にDuolingo風ポップUIにリデザインする。JSロジックは変更しない。

**Architecture:** 単一ファイル(index.html)内のCSSを全面書き換え。Google Fontsの読み込みをNunitoに変更。HTML構造はほぼ維持し、class名も維持。JSのSUIT_COLORS定数のみ色を調整。

**Tech Stack:** HTML/CSS (inline), Google Fonts (Nunito, JetBrains Mono)

---

### Task 1: フォントとCSS変数の刷新

**Files:**
- Modify: `index.html:7-8` (Google Fonts link)
- Modify: `index.html:10-29` (CSS変数)

**Step 1: Google Fontsリンクを変更**

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800;900&family=JetBrains+Mono:wght@600;700&display=swap" rel="stylesheet">
```

**Step 2: CSS変数を全面更新**

```css
:root {
  --bg: #f7f7f7;
  --bg-card: #ffffff;
  --bg-card-border: #e5e5e5;
  --surface: #f0f0f0;
  --text: #3c3c3c;
  --text-dim: #afafaf;
  --text-bright: #1a1a1a;
  --green: #58cc02;
  --green-dark: #46a302;
  --green-bright: #58cc02;
  --red: #ff4b4b;
  --red-dark: #cc3c3c;
  --red-bright: #ff4b4b;
  --blue: #1cb0f6;
  --blue-dark: #1899d6;
  --purple: #ce82ff;
  --purple-dark: #a568cc;
  --orange: #ff9600;
  --orange-dark: #cc7800;
  --gold: #ff9600;
  --gold-light: #ffb84d;
  --felt: #e8f5e9;
  --felt-light: #f1f8e9;
  --felt-dark: #c8e6c9;
}
```

**Step 3: ブラウザで開いて変数変更を確認**

Run: `open index.html`

**Step 4: Commit**

```
git add index.html
git commit -m "style: CSS変数とフォントをDuolingo風ポップUIに変更"
```

---

### Task 2: body・背景・基本スタイルの変更

**Files:**
- Modify: `index.html:31-71` (body, ::before, ::after)

**Step 1: body・ノイズ・グローを書き換え**

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Nunito', -apple-system, sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow-x: hidden;
  padding-bottom: 72px;
  position: relative;
}

/* ノイズテクスチャ・アンビエントグロー廃止 */
```

body::beforeとbody::afterのルールを完全削除する。

**Step 2: Commit**

```
git add index.html
git commit -m "style: body背景をライトグレーに変更、テクスチャ効果を廃止"
```

---

### Task 3: タブバーのリデザイン

**Files:**
- Modify: `index.html:73-140` (tab-bar CSS)

**Step 1: タブバーを白背景ポップスタイルに変更**

```css
.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 64px;
  background: #ffffff;
  border-top: 2px solid #e5e5e5;
  display: flex;
  justify-content: center;
  align-items: stretch;
  z-index: 200;
  padding-bottom: env(safe-area-inset-bottom);
  transition: transform 0.3s ease;
}
.tab-bar.hidden {
  transform: translateY(100%);
  pointer-events: none;
}
.tab-bar-inner {
  display: flex;
  width: 100%;
  max-width: 480px;
}
.tab-btn {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  background: none;
  border: none;
  color: var(--text-dim);
  font-family: 'Nunito', sans-serif;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.5px;
  cursor: pointer;
  transition: color 0.3s;
  -webkit-tap-highlight-color: transparent;
  position: relative;
}
.tab-btn .tab-icon {
  font-size: 20px;
  line-height: 1;
  transition: transform 0.3s, color 0.3s;
}
.tab-btn.active {
  color: var(--green);
}
.tab-btn.active .tab-icon {
  transform: scale(1.15);
}
.tab-btn.active::after {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 24px;
  height: 3px;
  background: var(--green);
  border-radius: 0 0 3px 3px;
}
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: タブバーを白背景ポップスタイルにリデザイン"
```

---

### Task 4: スタッツ・正解率リングのリデザイン

**Files:**
- Modify: `index.html:162-238` (stats, accuracy-ring CSS)

**Step 1: スタッツを白カード+シャドウに変更**

```css
.stats {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 14px 0 12px;
  margin-bottom: 6px;
  position: relative;
}
.stat-card {
  background: var(--bg-card);
  border: 2px solid var(--bg-card-border);
  border-radius: 16px;
  padding: 10px 14px;
  text-align: center;
  min-width: 0;
  flex: 1;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
}
.stat-card .stat-value {
  font-family: 'JetBrains Mono', monospace;
  font-size: 20px;
  font-weight: 700;
  color: var(--text-bright);
  line-height: 1.2;
}
.stat-card .stat-label {
  font-size: 9px;
  color: var(--text-dim);
  text-transform: uppercase;
  letter-spacing: 1.2px;
  margin-top: 3px;
  font-weight: 700;
}
.stat-card.accent {
  border-color: rgba(255,150,0,0.3);
  background: rgba(255,150,0,0.06);
}
.stat-card.accent .stat-value { color: var(--orange); }

.accuracy-ring-wrap {
  position: relative;
  width: 56px;
  height: 56px;
  flex-shrink: 0;
}
.accuracy-ring-wrap svg {
  transform: rotate(-90deg);
  width: 56px;
  height: 56px;
}
.accuracy-ring-wrap .ring-bg {
  fill: none;
  stroke: #e5e5e5;
  stroke-width: 3.5;
}
.accuracy-ring-wrap .ring-fg {
  fill: none;
  stroke: var(--green);
  stroke-width: 3.5;
  stroke-linecap: round;
  transition: stroke-dashoffset 0.8s cubic-bezier(0.22, 1, 0.36, 1);
  filter: drop-shadow(0 0 4px rgba(88,204,2,0.3));
}
.accuracy-ring-wrap .ring-text {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'JetBrains Mono', monospace;
  font-size: 12px;
  font-weight: 700;
  color: var(--text-bright);
}
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: スタッツカードを白背景+ソフトシャドウにリデザイン"
```

---

### Task 5: クイズフィルター・テーブルのリデザイン

**Files:**
- Modify: `index.html:240-301` (quiz-filter, poker-table CSS)

**Step 1: クイズフィルターをポップに**

```css
.quiz-filter {
  width: 100%;
  text-align: center;
  margin-bottom: 8px;
  font-size: 13px;
  color: var(--text);
  letter-spacing: 0.3px;
  font-weight: 600;
}
.quiz-filter select {
  background: var(--bg-card);
  color: var(--text);
  border: 2px solid var(--bg-card-border);
  padding: 6px 14px;
  border-radius: 10px;
  font-family: 'Nunito', sans-serif;
  font-size: 13px;
  font-weight: 600;
  -webkit-appearance: none;
}
```

**Step 2: ポーカーテーブルをシンプルポップに**

```css
.table-wrap {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-top: 14px;
  margin-bottom: 28px;
}
.poker-table {
  position: relative;
  width: 300px;
  height: 180px;
  max-width: calc(100vw - 48px);
}
.poker-table .felt {
  position: absolute;
  top: 24px;
  left: 24px;
  right: 24px;
  bottom: 24px;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  border-radius: 50%;
  border: 3px solid #a5d6a7;
  box-shadow: 0 4px 16px rgba(0,0,0,0.08);
}
/* フェルトテクスチャ廃止 */
.poker-table .felt::before {
  display: none;
}
```

**Step 3: Commit**

```
git add index.html
git commit -m "style: テーブルをシンプルなパステルグリーン楕円にリデザイン"
```

---

### Task 6: 座席バッジのリデザイン

**Files:**
- Modify: `index.html:302-367` (seat CSS)

**Step 1: 座席をポップなバッジスタイルに**

```css
.poker-table .seat {
  position: absolute;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Nunito', sans-serif;
  font-size: 9px;
  font-weight: 800;
  color: var(--text-dim);
  background: #ffffff;
  border: 2px solid #e5e5e5;
  transform: translate(-50%, -50%);
  transition: all 0.35s cubic-bezier(0.22, 1, 0.36, 1);
  line-height: 1.1;
  text-align: center;
  letter-spacing: 0.3px;
  overflow: visible;
  box-shadow: 0 2px 6px rgba(0,0,0,0.06);
}
.poker-table .seat.hero {
  background: var(--green);
  color: #fff;
  border-color: var(--green-dark);
  box-shadow:
    0 4px 12px rgba(88,204,2,0.35),
    0 0 0 3px rgba(88,204,2,0.15);
  z-index: 2;
  width: 46px;
  height: 46px;
  font-size: 10px;
  animation: heroPulse 3s ease-in-out infinite;
}
@keyframes heroPulse {
  0%, 100% { box-shadow: 0 4px 12px rgba(88,204,2,0.35), 0 0 0 3px rgba(88,204,2,0.15); }
  50% { box-shadow: 0 4px 16px rgba(88,204,2,0.5), 0 0 0 5px rgba(88,204,2,0.1); }
}
.poker-table .seat.behind {
  background: #f0f0f0;
  color: #999;
  border-color: #e0e0e0;
}
.poker-table .seat.acted {
  background: #f5f5f5;
  color: #ccc;
  border-color: #eee;
}
.poker-table .seat.waiting {
  background: #ffffff;
  color: var(--text-dim);
  border-color: #e5e5e5;
}
.poker-table .seat .fold-text {
  position: absolute;
  bottom: -12px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 7px;
  font-weight: 800;
  color: var(--red);
  opacity: 0;
  transition: opacity 0.15s;
  white-space: nowrap;
}
.poker-table .seat.fold-shown .fold-text { opacity: 1; }
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: 座席バッジを白背景ポップスタイルにリデザイン"
```

---

### Task 7: 対戦相手カード・エクイティ・チップのリデザイン

**Files:**
- Modify: `index.html:368-581` (opponent-cards, equity, chip, dealer-btn CSS)

**Step 1: 対戦相手カードをポップに**

opponent-cards, mini-card はほぼそのまま。mini-card のbox-shadowを軽くする。

```css
.mini-card {
  /* ...既存のまま... */
  box-shadow: 0 1px 4px rgba(0,0,0,0.15);
}
```

**Step 2: エクイティ表示はそのまま維持**

色のみ調整:
```css
.equity-fill.hero { background: var(--blue); }
.equity-fill.opp { background: var(--red); }
.equity-hero { color: var(--blue); }
.equity-opp { color: var(--red); }
```

**Step 3: チップをシンプル円形に**

```css
.poker-table .chip-disc {
  --chip-main: #b0bec5;
  --chip-dark: #78909c;
  --chip-light: #eceff1;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  position: relative;
  border: 2px solid var(--chip-dark);
  background: var(--chip-main);
  box-shadow: 0 2px 4px rgba(0,0,0,0.15);
}
/* テクスチャ系の疑似要素を非表示 */
.poker-table .chip-disc::before { display: none; }
.poker-table .chip-disc::after { display: none; }

.poker-table .chip-stack[data-type="blind"] .chip-disc {
  --chip-main: #90a4ae;
  --chip-dark: #607d8b;
}
.poker-table .chip-stack[data-type="raise"] .chip-disc {
  --chip-main: var(--red);
  --chip-dark: var(--red-dark);
}
.poker-table .chip-stack[data-type="hero"] .chip-disc {
  --chip-main: var(--orange);
  --chip-dark: var(--orange-dark);
}

.poker-table .chip-label {
  font-family: 'JetBrains Mono', monospace;
  font-size: 7.5px;
  font-weight: 700;
  color: var(--text-bright);
  white-space: nowrap;
  line-height: 1;
  margin-top: 2px;
  background: rgba(255,255,255,0.85);
  padding: 1.5px 4px;
  border-radius: 4px;
  letter-spacing: 0.3px;
  border: 1px solid #e5e5e5;
}
```

**Step 4: Dボタンをポップに**

```css
.poker-table .dealer-btn {
  position: absolute;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #ffffff;
  color: var(--text-bright);
  font-family: 'Nunito', sans-serif;
  font-size: 9px;
  font-weight: 900;
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translate(-50%, -50%);
  z-index: 4;
  border: 2px solid var(--green);
  box-shadow: 0 2px 6px rgba(88,204,2,0.25);
  letter-spacing: 0;
  line-height: 1;
}
```

**Step 5: hero-label をポップに**

```css
.poker-table .hero-label {
  position: absolute;
  bottom: -11px;
  left: 50%;
  transform: translateX(-50%);
  font-family: 'Nunito', sans-serif;
  font-size: 8px;
  color: #fff;
  font-weight: 800;
  white-space: nowrap;
  background: var(--green);
  padding: 1px 6px;
  border-radius: 4px;
  letter-spacing: 1.5px;
  border: none;
}
```

**Step 6: opener座席をポップに**

```css
.poker-table .seat.opener {
  background: var(--red);
  color: #fff;
  border-color: var(--red-dark);
  box-shadow: 0 4px 12px rgba(255,75,75,0.3);
  overflow: visible;
}
```

**Step 7: Commit**

```
git add index.html
git commit -m "style: チップ・Dボタン・対戦相手表示をポップスタイルにリデザイン"
```

---

### Task 8: トランプカードのリデザイン

**Files:**
- Modify: `index.html:582-655` (playing-card CSS)

**Step 1: カードをポップに**

```css
.playing-cards {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-bottom: 24px;
  perspective: 600px;
}
.playing-card {
  width: 82px;
  height: 116px;
  background: #ffffff;
  border-radius: 12px;
  position: relative;
  box-shadow:
    0 4px 12px rgba(0,0,0,0.1),
    0 1px 3px rgba(0,0,0,0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #e5e5e5;
  opacity: 0;
  animation: cardDeal 0.4s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}
/* 内側ボーダーを廃止 */
.playing-card::before { display: none; }
```

角丸rank/suitのフォントはNunitoに:

```css
.playing-card .corner-rank {
  font-family: 'Nunito', sans-serif;
  font-size: 18px;
  font-weight: 900;
  display: block;
}
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: トランプカードをフラットポップにリデザイン"
```

---

### Task 9: ボタンをDuolingo風3Dボタンにリデザイン

**Files:**
- Modify: `index.html:657-735` (btn CSS)

**Step 1: 3Dボタンスタイルに全面変更**

```css
.btn-row {
  display: flex;
  gap: 14px;
  width: 100%;
  margin-bottom: 16px;
}
.btn {
  flex: 1;
  padding: 16px;
  font-family: 'Nunito', sans-serif;
  font-size: 18px;
  font-weight: 800;
  border: none;
  border-radius: 16px;
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
  letter-spacing: 2px;
  position: relative;
  overflow: hidden;
  color: #fff;
  text-shadow: 0 1px 2px rgba(0,0,0,0.15);
  transition: filter 0.1s;
}
/* Glass highlight 廃止 */
.btn::after { display: none; }
.btn:active {
  transform: translateY(3px);
}
.btn:active {
  border-bottom-width: 0px;
  margin-bottom: 4px;
}
.btn:disabled { opacity: 0.35; cursor: default; transform: none; }

.btn-raise {
  background: var(--green);
  border-bottom: 4px solid var(--green-dark);
  box-shadow: 0 4px 12px rgba(88,204,2,0.3);
}
.btn-raise:active {
  background: var(--green);
  border-bottom: 0px solid var(--green-dark);
  transform: translateY(4px);
  box-shadow: none;
}

.btn-fold {
  background: var(--red);
  border-bottom: 4px solid var(--red-dark);
  box-shadow: 0 4px 12px rgba(255,75,75,0.3);
}
.btn-fold:active {
  background: var(--red);
  border-bottom: 0px solid var(--red-dark);
  transform: translateY(4px);
  box-shadow: none;
}

.btn-3bet {
  background: var(--purple);
  border-bottom: 4px solid var(--purple-dark);
  box-shadow: 0 4px 12px rgba(206,130,255,0.3);
}
.btn-3bet:active {
  background: var(--purple);
  border-bottom: 0px solid var(--purple-dark);
  transform: translateY(4px);
  box-shadow: none;
}

.btn-call {
  background: var(--blue);
  border-bottom: 4px solid var(--blue-dark);
  box-shadow: 0 4px 12px rgba(28,176,246,0.3);
}
.btn-call:active {
  background: var(--blue);
  border-bottom: 0px solid var(--blue-dark);
  transform: translateY(4px);
  box-shadow: none;
}
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: ボタンをDuolingo風3D押し込みボタンにリデザイン"
```

---

### Task 10: フィードバック・結果表示のリデザイン

**Files:**
- Modify: `index.html:737-837` (last-result, flash, result-symbol CSS)

**Step 1: 結果バーをポップに**

```css
.last-result-inner.correct {
  background: rgba(88,204,2,0.12);
  color: var(--green);
  border: 2px solid rgba(88,204,2,0.25);
}
.last-result-inner.wrong {
  background: rgba(255,75,75,0.12);
  color: var(--red);
  border: 2px solid rgba(255,75,75,0.25);
}
```

**Step 2: フラッシュをポップカラーに**

```css
#quiz-screen.flash-correct::before {
  background: radial-gradient(ellipse at 50% 60%, rgba(88,204,2,0.2), transparent 65%);
  ...
}
#quiz-screen.flash-wrong::before {
  background: radial-gradient(ellipse at 50% 60%, rgba(255,75,75,0.2), transparent 65%);
  ...
}
```

**Step 3: 結果シンボルをポップに**

```css
.result-symbol.correct { color: var(--green); }
.result-symbol.wrong { color: var(--red); }
```

**Step 4: Commit**

```
git add index.html
git commit -m "style: フィードバック表示をポップカラーにリデザイン"
```

---

### Task 11: リセットボタン・キーボードヒント・レンジ表のリデザイン

**Files:**
- Modify: `index.html:838-952` (reset-btn, kb-hint, range CSS)

**Step 1: リセットボタン・ヒントをポップに**

```css
.reset-btn {
  ...既存のレイアウトは維持...
  color: var(--text-dim);
  ...
}
.reset-btn:hover { color: var(--text); }

.kb-hint {
  font-size: 10px;
  color: var(--text-dim);
  ...
}
```

**Step 2: レンジ表をポップに**

```css
.range-header h2 {
  font-family: 'Nunito', sans-serif;
  font-size: 22px;
  font-weight: 800;
  color: var(--text-bright);
}
.position-select {
  background: var(--bg-card);
  color: var(--text);
  border: 2px solid var(--bg-card-border);
  ...
  font-family: 'Nunito', sans-serif;
  ...
}
.range-cell {
  ...
  font-family: 'JetBrains Mono', monospace;
  ...
  border-radius: 4px;
  ...
}
.range-cell.in-range {
  outline: 2px solid var(--orange);
  ...
}
.range-cell.fold-cell {
  background: #f0f0f0 !important;
  color: #ccc;
}
.legend-color {
  width: 12px;
  height: 12px;
  border-radius: 4px;
}
```

**Step 3: Commit**

```
git add index.html
git commit -m "style: レンジ表・補助UIをポップスタイルにリデザイン"
```

---

### Task 12: ログ画面のリデザイン

**Files:**
- Modify: `index.html:954-1070` (log CSS)

**Step 1: ログ画面をポップに**

```css
.log-header h2 {
  font-family: 'Nunito', sans-serif;
  font-size: 22px;
  font-weight: 800;
  color: var(--text-bright);
}
.log-filter select {
  ...
  font-family: 'Nunito', sans-serif;
  border: 2px solid var(--bg-card-border);
  ...
}
.log-entry {
  ...
  border-bottom: 1px solid #f0f0f0;
  ...
}
.log-answer.raise { background: rgba(88,204,2,0.12); color: var(--green); }
.log-answer.call { background: rgba(28,176,246,0.12); color: var(--blue); }
.log-answer.fold { background: rgba(255,75,75,0.12); color: var(--red); }
.log-scenario.sc-open { background: rgba(88,204,2,0.12); color: var(--green); }
.log-scenario.sc-vs-open { background: rgba(28,176,246,0.12); color: var(--blue); }
.log-scenario.sc-vs-3bet { background: rgba(255,150,0,0.12); color: var(--orange); }
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: ログ画面をポップスタイルにリデザイン"
```

---

### Task 13: レスポンシブ調整

**Files:**
- Modify: `index.html:1072-1089` (media queries)

**Step 1: 既存のmedia queryを維持しつつフォント名を更新**

座席のfont-family等を`'Nunito'`に変更。

**Step 2: Commit**

```
git add index.html
git commit -m "style: レスポンシブ調整のフォント名を更新"
```

---

### Task 14: JS側の色定数を更新

**Files:**
- Modify: `index.html:1833` (SUIT_COLORS)

**Step 1: スート色をポップに**

```javascript
const SUIT_COLORS = { s: '#3c3c3c', h: '#ff4b4b', d: '#1cb0f6', c: '#58cc02' };
```

**Step 2: Commit**

```
git add index.html
git commit -m "style: スートカラーをポップパレットに変更"
```

---

### Task 15: 最終確認とプッシュ

**Step 1: ブラウザで全画面動作確認**

Run: `open index.html`

確認項目:
- クイズ画面: テーブル・カード・ボタン表示
- RAISE/FOLD の正解/不正解フィードバック
- vs オープンの3ボタン表示
- レンジ表画面
- ログ画面
- モバイルサイズ(DevTools)

**Step 2: 問題なければプッシュ**

```
git push origin main
```

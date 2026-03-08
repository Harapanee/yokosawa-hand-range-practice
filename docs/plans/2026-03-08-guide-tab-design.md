# ガイドタブ追加 Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 4つ目のタブ「ガイド」を追加し、アプリの説明・使い方・判定ルールを表示する

**Architecture:** 既存の `.screen` パターンに従い `#guide-screen` を追加。アコーディオン形式で3セクションを折りたたみ表示。タブバーに4つ目のボタンを追加し `showGuide()` で切り替え。

**Tech Stack:** Vanilla HTML/CSS/JS（既存と同じ）

---

### Task 1: ガイド画面のCSS追加

**Files:**
- Modify: `index.html` (CSS セクション、`</style>` 直前に追加)

**Step 1: アコーディオンとガイド画面用CSSを追加**

`/* ===== SMALL SCREEN ===== */` の直前に以下を挿入:

```css
/* ===== GUIDE SCREEN ===== */
.guide-header {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 14px;
}
.guide-header h2 {
  font-family: 'Nunito', sans-serif;
  font-size: 22px;
  font-weight: 800;
  color: var(--text-bright);
}
.guide-section {
  width: 100%;
  background: var(--bg-card);
  border: 2px solid var(--bg-card-border);
  border-radius: 14px;
  margin-bottom: 10px;
  overflow: hidden;
}
.guide-toggle {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  background: none;
  border: none;
  cursor: pointer;
  font-family: 'Nunito', sans-serif;
  font-size: 15px;
  font-weight: 700;
  color: var(--text-bright);
  -webkit-tap-highlight-color: transparent;
}
.guide-toggle .guide-arrow {
  font-size: 12px;
  color: var(--text-dim);
  transition: transform 0.3s;
}
.guide-section.open .guide-arrow {
  transform: rotate(180deg);
}
.guide-body {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.35s ease;
}
.guide-section.open .guide-body {
  max-height: 2000px;
}
.guide-content {
  padding: 0 16px 16px;
  font-size: 13px;
  line-height: 1.7;
  color: var(--text);
}
.guide-content h3 {
  font-size: 14px;
  font-weight: 700;
  color: var(--text-bright);
  margin: 14px 0 6px;
}
.guide-content h3:first-child {
  margin-top: 0;
}
.guide-content p {
  margin: 6px 0;
}
.guide-content ul {
  margin: 6px 0;
  padding-left: 20px;
}
.guide-content li {
  margin: 3px 0;
}
.guide-content .tier-table {
  width: 100%;
  border-collapse: collapse;
  margin: 10px 0;
  font-size: 12px;
}
.guide-content .tier-table th,
.guide-content .tier-table td {
  padding: 6px 8px;
  border: 1px solid var(--bg-card-border);
  text-align: left;
}
.guide-content .tier-table th {
  background: var(--surface);
  font-weight: 700;
  font-size: 11px;
  color: var(--text-dim);
}
.guide-content .tier-badge {
  display: inline-block;
  width: 14px;
  height: 14px;
  border-radius: 3px;
  vertical-align: middle;
  margin-right: 4px;
}
.guide-content .rule-box {
  background: var(--surface);
  border-radius: 8px;
  padding: 10px 14px;
  margin: 8px 0;
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  line-height: 1.8;
  white-space: pre-wrap;
}
.guide-content .key-badge {
  display: inline-block;
  background: var(--surface);
  border: 1px solid var(--bg-card-border);
  border-radius: 4px;
  padding: 1px 6px;
  font-family: 'JetBrains Mono', monospace;
  font-size: 11px;
  font-weight: 700;
}
```

---

### Task 2: ガイド画面のHTML追加

**Files:**
- Modify: `index.html` (`<!-- Bottom Tab Bar -->` の直前に追加)

**Step 2: ガイド画面HTMLを挿入**

`</div>` (log-screen閉じ) と `<!-- Bottom Tab Bar -->` の間に `#guide-screen` を追加。
3つのアコーディオンセクション:
1. このアプリについて
2. 使い方
3. 判定ルール（ティア表、シナリオ別ロジック、BB特殊ルール）

ティアのカラーバッジは JS の `TIER_COLORS` と同じ値をインラインで使う。

---

### Task 3: タブバーに4つ目のボタン追加

**Files:**
- Modify: `index.html` (タブバーHTML)

**Step 3: ガイドタブボタンを追加**

```html
<button class="tab-btn" id="tab-guide" onclick="showGuide()">
  <span class="tab-icon">?</span>
  <span>ガイド</span>
</button>
```

---

### Task 4: JS に画面切り替えロジック追加

**Files:**
- Modify: `index.html` (SCREEN SWITCHING セクション)

**Step 4a: `hideAllScreens()` に guide-screen を追加**

```js
document.getElementById('guide-screen').classList.remove('active');
```

**Step 4b: `showGuide()` 関数を追加**

```js
function showGuide() {
  hideAllScreens();
  document.getElementById('guide-screen').classList.add('active');
  document.getElementById('tab-guide').classList.add('active');
}
```

**Step 4c: アコーディオン開閉関数を追加**

```js
function toggleGuideSection(btn) {
  btn.closest('.guide-section').classList.toggle('open');
}
```

---

### Task 5: ブラウザ確認 → コミット → プッシュ

**Step 5a:** `open index.html` で動作確認
**Step 5b:** コミット＆プッシュ

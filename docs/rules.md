# ヨコサワ・トーナメント用ハンドレンジ ルール仕様書

このドキュメントは、アプリ (`index.html`) の正誤判定ロジックが準拠するルールを記述したものです。

---

## 1. ティア（色）の定義

ハンドは強さ順に T1〜T8 の8段階に分類される。T0 はどのティアにも属さないフォールド専用ハンド。

| ティア | 色名 | 意味 | ハンド一覧 |
|--------|------|------|-----------|
| T1 | 紺 | プレミア | AA, KK, QQ, AKs, AKo |
| T2 | 赤 | 8人中 | AQs, AJs, ATs, KQs, AQo, JJ, TT, 99 |
| T3 | 黄 | 8人弱 | KJs, KQo, AJo, QJs, JTs, 88, 77 |
| T4 | 緑 | 6〜7人 | A9s-A2s, KTs, K9s, QTs, KJo, ATo, T9s, 66, 55 |
| T5 | 水色 | 4〜5人 | Q9s, J9s, T8s, 98s, QJo, KTo, JTo, A9o, 44, 33, 22 |
| T6 | 白 | 3人 | K8s-K2s, Q8s-Q6s, J8s, J7s, 97s, 87s, 76s, 65s, K9o, Q9o, J9o, T9o, QTo, A8o, A7o |
| T7 | 紫 | 2人 | Q5s-Q2s, J6s, T7s, 96s, 86s, 75s, 64s, 54s, 98o, A6o |
| T8 | ピンク | BB対BTN専用 | J5s-J2s, T6s-T3s, 95s, 85s, 74s, 63s, 53s, 43s, K8o, Q8o, J8o, T8o, K7o, Q7o, 97o, 87o, A5o-A2o, K6o, K5o |
| T0 | グレー | フォールド | 上記に含まれない全ハンド |

> **ティア番号が小さいほど強い。** T1 が最強、T8 が最弱。

---

## 2. ポジションとオープンレンジ

9人テーブル。各ポジションの `maxTier` が「そのポジションで参加できる最も弱いティア」を示す。

| ポジション | 後ろ人数 | maxTier | オープン範囲 |
|-----------|---------|---------|------------|
| UTG | 8 | T3 | T1〜T3 |
| UTG+1 | 7 | T4 | T1〜T4 |
| MP | 6 | T4 | T1〜T4 |
| LJ | 5 | T5 | T1〜T5 |
| HJ | 4 | T5 | T1〜T5 |
| CO | 3 | T6 | T1〜T6 |
| BTN | 2 | T7 | T1〜T7 |
| SB | 1 | T7 | T1〜T7 |
| BB | 0 | - | オープン不可（防御専用） |

> **読み方:** 「後ろに人が多い＝厳しく」「少ない＝広く参加OK」

---

## 3. シナリオ別の正解判定

### 3-1. オープン（自分が最初にレイズ）

```
ハンドのティア ≦ ポジションのmaxTier → RAISE
それ以外                              → FOLD
```

**例:**
- HJ(maxTier=T5) で AJo(T3) → T3 ≦ T5 → **RAISE**
- UTG(maxTier=T3) で A9s(T4) → T4 > T3 → **FOLD**
- BTN(maxTier=T7) で J6s(T7) → T7 ≦ T7 → **RAISE**

---

### 3-2. vs オープン（前の人のレイズに対してどうするか）

相手（オープナー）の `maxTier` を基準に、自分のハンドのティアと比較する。

#### 非BBの場合

```
ハンドのティア ≦ openerMax - 2  → 3BET（2色上以上）
ハンドのティア  = openerMax - 1  → CALL（1色上）
それ以外                         → FOLD（同色以下）
```

**考え方:** 相手のオープンレンジの最弱ティアを基準に、
- 同じ色 → フォールド
- 1つ上の色 → コール
- 2つ以上上の色 → 3BET（リレイズ）

**例（COオープン、maxTier=T6 に対して）:**
- BTNで T9s(T4) → T4 ≦ T6-2=T4 → **3BET**
- BTNで Q9s(T5) → T5 = T6-1=T5 → **CALL**
- BTNで Q8s(T6) → T6 > T5 → **FOLD**（同色）

**例（UTGオープン、maxTier=T3 に対して）:**
- MPで AKs(T1) → T1 ≦ T3-2=T1 → **3BET**
- MPで AQs(T2) → T2 = T3-1=T2 → **CALL**
- MPで KJs(T3) → FOLD（同色）

#### BBの場合（防御が広い）

3BET の閾値は非BBと同じ: `tier ≦ openerMax - 2`

**CALL 範囲が拡張される:**

| オープナー | CALL上限 | 含まれる追加ティア |
|-----------|---------|-----------------|
| BTN | T8まで | 紫 + ピンクまで |
| CO | T7まで | 紫まで |
| それ以外 | T6まで | 白まで（基本防衛） |

```
ハンドのティア ≦ openerMax - 2  → 3BET
ハンドのティア ≦ callMax        → CALL
それ以外                         → FOLD
```

**例（BTNオープン vs BB）:**
- 87o(T8) → callMax=T8 → T8 ≦ T8 → **CALL**
- 86o(T0) → ティアなし → **FOLD**

**例（UTGオープン vs BB）:**
- AQs(T2) → T2 ≦ T3-2=T1? No → T2 ≦ T6? Yes → **CALL**
- KJs(T3) → T3 ≦ T1? No → T3 ≦ T6? Yes → **CALL**
- K8s(T6) → T6 ≦ T1? No → T6 ≦ T6? Yes → **CALL**
- Q5s(T7) → T7 ≦ T1? No → T7 ≦ T6? No → **FOLD**

> **BBのポイント:** 既に1BB払っているので参加コストが低く、広く守る。「白(T6)以上は基本コール」が最低ライン。CO/BTN相手はさらに拡張。

---

### 3-3. vs 3bet（自分がオープンした後、後ろから3betされた）

自分（オープナー）の `maxTier` を基準にする。

```
ハンドのティア  = T1              → 4BET（常に）
ハンドのティア ≦ heroMax - 4     → 4BET
ハンドのティア ≦ heroMax - 2     → CALL
それ以外                          → FOLD
```

**考え方:** 3betする相手のレンジは「自分のmaxTierの2色上」と想定。その相手に対して再び「2色上=4BET、同色=CALL」の原則を適用。色が1段ずつ上がっていくイメージ。

- **T1（AA, KK, QQ, AKs, AKo）は何があっても降りない**

| ポジション | maxTier | 4BET | CALL | FOLD |
|-----------|---------|------|------|------|
| UTG | T3 | T1 | - | T2〜T3 |
| UTG+1 | T4 | T1 | T2 | T3〜T4 |
| MP | T4 | T1 | T2 | T3〜T4 |
| LJ | T5 | T1 | T2〜T3 | T4〜T5 |
| HJ | T5 | T1 | T2〜T3 | T4〜T5 |
| CO | T6 | T1〜T2 | T3〜T4 | T5〜T6 |
| BTN | T7 | T1〜T3 | T4〜T5 | T6〜T7 |
| SB | T7 | T1〜T3 | T4〜T5 | T6〜T7 |

**例（BTN maxTier=T7 が3betされた）:**
- AKs(T1) → T1 = T1 → **4BET**
- KJs(T3) → T3 ≦ T7-4=T3 → **4BET**
- A9s(T4) → T4 ≦ T7-2=T5 → **CALL**
- Q9s(T5) → T5 ≦ T5 → **CALL**
- K8s(T6) → T6 > T5 → **FOLD**

---

## 4. チェーンシステム（オープン → vs 3bet）

アプリ独自の仕組み。オープンで正解RAISEした後、40%の確率で後ろのプレイヤーが3betしてくる演出が発生する。

1. ヒーローが RAISE（正解）
2. 40% の確率で後ろの誰かが 3bet
3. そのまま vs 3bet の問題に移行（ハンドは変わらない）
4. 3bettor のハンドは `T1〜(heroMax - 2)` からランダム選出

---

## 5. まとめ：判定フロー早見表

```
                    ┌─ RAISE (tier ≦ maxTier)
   オープン ────────┤
                    └─ FOLD

                    ┌─ 3BET  (tier ≦ openerMax - 2)
   vs オープン ─────┼─ CALL  (tier = openerMax - 1)  ※BB: callMaxまで拡張
                    └─ FOLD

                    ┌─ 4BET  (tier = 1 or tier ≦ heroMax - 4)
   vs 3bet ─────────┼─ CALL  (tier ≦ heroMax - 2)
                    └─ FOLD
```

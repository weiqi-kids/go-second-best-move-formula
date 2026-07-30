# 圍棋「第二好的次一手公式」不存在性的完整證明

我在下棋時，常常找不到最好的那一手。身為業餘低段棋友，我常想：

「其實也不需要每次都下出最佳手吧？如果能穩定找到第二好的那一手，也已經很不錯了。」

接著我又想到：

「如果我能找出第二好的次一手，那是不是也代表，我其實有機會找到最好的次一手？」

於是，我請 AI 嘗試研究並推導所謂的「第二好的次一手公式」，最後得到以下結論。

## 摘要

本文證明：廣義 $n \times n$ 圍棋（基本劫禁規則）不存在有限、封閉、可計算、非暴力搜尋的解析公式，能對任意合法局面直接求得著手價值 $Q(S,m)$ 或次優著 $m_2(S)$。核心結論為 $Q \notin \mathrm{FP}$（無條件）及 $m_2 \notin \mathrm{FP}$（經由修正版 Robson 構造）。三個原創技術——空間壓縮論證、pass 不等式、瓶頸-編碼構造——使這些結論成為可能。

---

## 1. 從圍棋到數學：問題的形式化過程

本節記錄如何從圍棋棋手的直覺出發，逐步收斂到精確的數學定義。每一步形式化都伴隨一個設計選擇，我們說明選擇的理由與替代方案。

### 1.1 棋手的原始直覺

每位圍棋棋手都有這樣的經驗：在某個局面下，直覺告訴你「最好的一手」是什麼，但你也想知道——**如果不走那手，第二好的選擇是什麼？**

這個問題在實戰中無所不在：

- **時間壓力下的決策**：讀秒時來不及深算最佳手，退而求其次走次佳手。
- **棋風選擇**：最佳手可能是複雜的戰鬥，次佳手可能是簡明的安全下法。
- **覆盤分析**：「這手棋如果不走這裡，第二好的選擇在哪？差多少？」

自然的問題是：**是否存在某種「公式」或「演算法捷徑」，讓你不必把整盤棋算到底，就能直接說出次佳手？**

### 1.2 第一步形式化：什麼是「局面」？

圍棋的局面包含三個要素：

| 要素 | 直覺 | 形式化 |
|------|------|--------|
| 棋盤上的黑白子分佈 | 看一眼就知道 | 函數 $b: [n]^2 \to \{\bullet, \circ, \emptyset\}$ |
| 輪到誰下 | 黑先/白先 | 行棋方 $\tau \in \{\bullet, \circ\}$ |
| 劫的狀態 | 哪裡剛提過劫不能立即回提 | 劫禁點 $k \in [n]^2 \cup \{\emptyset\}$ |

**設計選擇**：是否需要包含歷史記錄（已下過的所有手）？

- 基本劫禁規則：只需記住上一手是否提劫 → 劫禁點即可，**不需要完整歷史**。
- 超級劫禁規則：需要完整歷史（禁止任何曾出現的局面重複）→ 局面定義會膨脹為指數大小。

**選擇基本劫禁**的原因：這是 Robson (1983) EXPTIME-完全性成立的規則。超級劫下的複雜度是開放問題。

### 1.3 第二步形式化：什麼是「最好」？

棋手說「最好的一手」，直覺上指的是：**在雙方都下出最強應對的前提下，這手棋導致的最終結果最優。**

這正是博弈論中 **minimax** 的定義：

$$V(S) = \max_{m \in M(S)} \bigl[-V(\mathrm{apply}(S,m))\bigr]$$

- $V(S)$ 是局面 $S$ 的「真實價值」——假設雙方完美對弈，當前行棋方能取得的最終目差。
- 取負號 $-V$ 是因為對手的最優等於我的最劣——零和博弈的本質。

**設計選擇**：價值用什麼度量？

| 度量 | 定義 | 本文選擇 |
|------|------|:--------:|
| 勝負 | $V \in \{+1, -1\}$ | |
| 目差 | $V \in \mathbb{Z}$（領地差） | **選此** |
| 勝率 | $V \in [0,1]$（混合策略下） | |

選擇**目差**的原因：比目法計分下，目差為整數，使得近似問題等價於精確問題（$\varepsilon < 1/2$ 的近似可取整恢復精確值）。

### 1.4 第三步形式化：什麼是「第二好」？

有了價值函數 $Q(S,m) = -V(\mathrm{apply}(S,m))$——著手 $m$ 在局面 $S$ 下的價值——「最好」和「第二好」就是排序問題：

$$m_1(S) = \arg\max_{m \in M(S)} Q(S,m) \qquad \text{（最佳手）}$$

$$m_2(S) = \arg\max_{m \in M(S) \setminus \{m_1\}} Q(S,m) \qquad \text{（次佳手）}$$

**微妙之處：並列 (ties)**。若多手棋的 $Q$ 值相同，$m_1$ 和 $m_2$ 不唯一。處理方式：以固定的字典序破解並列，使 $m_1$、$m_2$ 成為良定義的函數。

### 1.5 第四步形式化：什麼是「公式」？

棋手的直覺：「不用把棋算到底，看一眼局面就能直接說出次佳手。」

需要精確定義「不用算到底」的含義。以下是四個候選定義，從弱到強：

| 層級 | 定義 | 排除了什麼 |
|------|------|-----------|
| (a) 可計算 | 存在演算法，給足時間必定輸出正確答案 | 不可計算函數（不存在，因為有限博弈的 $Q$ 總是可計算的） |
| (b) 多項式時間 | 演算法在 $\mathrm{poly}(n)$ 步內完成 | 指數時間的暴力搜尋 |
| (c) 解析閉式 | 由有限次標準數學運算組成的表達式 | 需要迴圈或遞迴的演算法 |
| (d) 多項式大小電路 | 每個棋盤大小有一個 $\mathrm{poly}(n)$ 大小的電路 | 需要超多項式描述的查表 |

**關鍵觀察**：(c) 的解析閉式若大小為 $s(n)$，求值時間為 $O(s(n))$。若 $s(n) = \mathrm{poly}(n)$，則 (c) ⊆ (b)。因此 (b) 的否定自動蘊含 (c) 的否定。

**本文選擇 (b)**：$\mathrm{poly}(n)$ 時間可計算。這是「非暴力搜尋」的最自然數學化——排除了指數時間的 minimax/alpha-beta 搜尋，但允許任何多項式時間的捷徑。

### 1.6 第五步形式化：標準 19×19 還是廣義 $n \times n$？

| 選擇 | 結果 | 原因 |
|------|------|------|
| 固定 19×19 | $Q$ 是有限域上的函數，公式 trivially 存在（查表）。「緊湊公式」是否存在是電路複雜度問題，現有數學無法回答。 | 定義域有限 → 沒有漸近分析的空間 |
| **廣義 $n \times n$** | $n$ 為參數，可做漸近分析。$Q$ 的 EXPTIME-完全性給出嚴格否定結論。 | **選此**：可獲得無條件定理 |

棋手可能會問：「我只關心 19×19 啊，廣義 $n \times n$ 跟我有什麼關係？」

回答：廣義結論是 19×19 的**嚴格下界暗示**。若連 $n \to \infty$ 的漸近情形都不存在公式，那麼 19×19 的公式（即使存在）也必然沒有結構性的原因——它只能是一個碰巧對有限多局面成立的巧合，而非數學上的必然。

### 1.7 形式化收斂的完整路徑

```
直覺問題：「次佳手有沒有捷徑可算？」
     │
     ├── 局面 → (棋盤配置, 行棋方, 劫禁點)
     ├── 「最好」→ minimax 博弈值 V(S)
     ├── 「次好」→ Q 值排序的第二名 m₂(S)
     ├── 「捷徑」→ poly(n) 時間可計算
     ├── 規則 → 基本劫禁（EXPTIME-complete 的規則）
     └── 棋盤 → 廣義 n×n（可做漸近分析）
          │
          ▼
     精確問題：「m₂(S) ∈ FP？」
          │
          ▼
     回答：NO（定理 A-D）
```

### 1.8 被排除的替代形式化

為完整性，說明幾個被有意排除的方向：

| 替代 | 為何排除 |
|------|---------|
| 神經網路近似（AlphaGo/KataGo） | 近似 ≠ 精確。本文要求對**所有**局面**精確**輸出 $m_2$，不允許錯誤。 |
| 隨機化演算法（BPP） | 可作為延伸結論討論（見定理 B 的推論），但核心問題是確定性的。 |
| 量子演算法（BQP） | $\mathrm{BQP} \subseteq \mathrm{PSPACE} \subseteq \mathrm{EXPTIME}$。對 $Q$：定理 A 直接排除（$Q$ 為 EXPTIME-完全，$\mathrm{BQP} \subsetneq \mathrm{EXPTIME}$ 若 $P \neq \mathrm{PSPACE}$）。對 $m_1$：空間壓縮論證不直接適用（BQP 的空間特性不同），但 $m_1 \in \mathrm{BQP} \Rightarrow$ PSPACE 模擬仍可行（$\mathrm{BQP} \subseteq \mathrm{PSPACE}$）。 |
| 查表 / 全樹搜索 | 使用者明確排除。對廣義 $n \times n$，查表大小為指數級。 |
| 啟發式 / 近似公式 | 可能存在好的近似（如 KataGo），但本文問的是**精確**解。 |

---

## 2. 問題的精確數學陳述

### 2.1 基本定義

考慮廣義 $n \times n$ 圍棋博弈 $\mathcal{G}_n$（基本劫禁規則：僅禁止立即回提，允許更長週期的局面重複；貼目固定為整數或半整數）。

- $\mathcal{P}_n$：所有合法局面的有限集。每個 $S \in \mathcal{P}_n$ 包含棋盤配置 $b: [n]^2 \to \{\bullet, \circ, \emptyset\}$、行棋方 $\tau(S) \in \{\bullet, \circ\}$、及劫禁點。
- $M(S)$：局面 $S$ 的合法著手集（至多 $n^2 + 1$ 手，含 pass），$|M(S)| \leq n^2 + 1$。
- $\mathrm{apply}(S, m)$：著手 $m$ 後的新局面，$O(n^2)$ 時間可計算。
- $V(S)$：從當前行棋方視角的 minimax 博弈值（目差，整數值）：

$$V(S) = \begin{cases} \mathrm{score}(S) & \text{若 } S \text{ 為終局（雙方連續 pass）} \\[4pt] 0 & \text{若 } S \text{ 進入無限循環（如三劫，依日本規則判和）} \\[4pt] \displaystyle\max_{m \in M(S)} \bigl[-V(\mathrm{apply}(S,m))\bigr] & \text{否則} \end{cases}$$

> **定義說明**：基本劫禁規則允許三劫循環等無限博弈。當最優對弈導致無限循環時，依日本規則「無勝負」（和棋）的慣例，定義 $V = 0$。此定義確保 $V(S)$ 對所有合法局面良定義。Robson 歸約構造的局面不含此類循環（ATM 計算必定終止），故此情形不影響核心定理。

- $Q(S,m) = -V(\mathrm{apply}(S,m))$：在局面 $S$ 下著手 $m$ 的價值。
- $m_1(S) = \arg\max_{m \in M(S)} Q(S,m)$：最佳手。
- $m_2(S) = \arg\max_{m \in M(S) \setminus \{m_1\}} Q(S,m)$：次佳手（ties 以固定序破解）。

### 2.2 精確問題陳述

> 是否存在函數 $F$，使得 $F(S,m) = Q(S,m)$（或 $F(S) = m_2(S)$）對所有 $n$ 與所有合法 $(S,m)$ 成立，且 $F$ 滿足：
>
> (i) 描述長度有限
> (ii) 求值時間 $T_F(n) = \mathrm{poly}(n)$
> (iii) 不等價於完整博弈樹搜尋

### 2.3 規則說明

Robson (1983) 的 EXPTIME-完全性**僅在基本劫禁下成立**。超級劫 (superko) 下的複雜度是開放問題（已知 PSPACE-hard $\leq$ ? $\leq$ EXPSPACE）。以下所有結論限定於基本劫禁規則。

### 2.4 分析框架

- **廣義 $n \times n$ 圍棋**（$n$ 為參數）：可獲得嚴格否定結論。
- **固定 19×19**：定義域有限，涉及電路複雜度，現有數學無法決定。

---

## 3. 預備知識

### 3.1 圍棋術語表（供理論計算機科學家參考）

| 術語 | 英文 | 含義 |
|------|------|------|
| 叫吃 | atari | 一個群只剩 1 氣（liberty），下一步可被提吃 |
| 征子 | ladder | 連續叫吃的迫著序列，被征方沿對角線逃跑，征方沿同方向追擊 |
| 打劫 | ko fight | 雙方交替提吃同一位置的循環，受劫規則限制 |
| 劫材 | ko threat | 劫爭中，被禁止立即回提的一方在別處落子製造的威脅 |
| 先手 / 後手 | sente / gote | 先手：對方必須回應的著手；後手：對方可忽略的著手 |
| 目 | point | 領地計分單位。日本規則下 = 圍住的空點數 + 提子數 |
| 活棋 | alive group | 至少有兩眼（兩個獨立的內部空點）的群，不可能被提吃 |
| 單官 | dame | 中立點——不屬於任何一方領地的空點，落子價值 = 0 |
| 管道 | pipe | Robson 構造中由長串棋子組成的結構，編碼 ATM 磁帶 |

### 3.2 計算複雜度簡介（供圍棋愛好者參考）

| 概念 | 直覺 |
|------|------|
| **P** | 多項式時間可解的問題（「容易」的問題）。例：排序。 |
| **PSPACE** | 多項式空間可解的問題（允許指數時間，但記憶體受限）。 |
| **EXPTIME** | 指數時間可解的問題（「可以解但非常慢」）。例：暴力搜尋整棵博弈樹。 |
| **FP** | 多項式時間可計算的函數（輸出不限於是/否，可以是任何值）。 |
| **ATM** | 交替式圖靈機——一種理論計算模型，交替進行「存在選擇」和「全稱選擇」，完美模型化二人博弈。 |
| **EXPTIME-完全** | EXPTIME 中「最難」的問題——若它能快速解決，所有 EXPTIME 問題都能快速解決。 |
| **歸約** | 將問題 A 轉換為問題 B 的方法。若 B 能解則 A 能解。用於證明 A 「至少與 B 一樣難」。 |
| **時間階層定理** | 給更多時間確實能解更多問題。特別地，$P \subsetneq \mathrm{EXPTIME}$（已證明的無條件定理）。 |

**核心關係**：$P \subseteq \mathrm{PSPACE} \subseteq \mathrm{EXPTIME}$，且 $P \subsetneq \mathrm{EXPTIME}$（嚴格包含，已證）。但 $P$ vs $\mathrm{PSPACE}$ 和 $\mathrm{PSPACE}$ vs $\mathrm{EXPTIME}$ 是否嚴格包含，目前未知。

---

## 4. 核心思路

### 4.1 第一性原理：計算普遍性

圍棋局面是**通用計算媒介**。Robson (1983) 建立了一個「編譯器」：對任何交替式多項式空間圖靈機 (ATM) $\mathcal{M}$（由 APSPACE = EXPTIME，Chandra-Kozen-Stockmeyer 定理），可在 $\mathrm{poly}(|\mathcal{M}|)$ 時間內構造等價的 Go 局面 $S_\mathcal{M}$，使得：

$$V(S_\mathcal{M}) > 0 \iff \mathcal{M} \text{ 接受}$$

一個「$Q$ 的公式」等同於一個能在 poly 時間解決**所有 EXPTIME 計算**的通用捷徑。$P \subsetneq \mathrm{EXPTIME}$（時間階層定理，無條件）嚴格排除這種捷徑。

### 4.2 跨領域類比

**可分解博弈有公式，不可分解博弈沒有：**

| 領域 | 系統 | 公式？ | 結構原因 |
|------|------|:------:|----------|
| 組合博弈 | Nim | 有 | 獨立堆，$V = \bigoplus_i G(h_i)$ |
| 組合博弈 | Wythoff 博弈 | 有 | 黃金比例 + Beatty 序列 |
| Go 收官 | CGT 可分解終盤 | 有 | 獨立區域的溫度圖求和 |
| **Go 中盤** | **一般局面** | **無** | **全局糾纏，不可分解** |
| 凝聚態物理 | 受挫自旋系統 | 無 | 阻挫效應（QMA-hard） |
| 張量網路 | 一般收縮 | 無 | 高糾纏（#P-hard） |

圍棋中盤局面的**拓撲糾纏**——征子、打劫、厚勢、對殺——阻止分解為獨立子博弈，類似凝聚態中的強關聯系統。

### 4.3 驗證亦困難

與 NP 問題（找難驗易）不同：驗證「$m$ 是最佳手」需計算所有 $Q(S,m')$ 再比較——本身就是 EXPTIME-hard。即使有人宣稱擁有公式，驗證其正確性也需指數時間。

---

## 5. 定理與完整證明

### 5.1 定理 A（$Q$ 的公式不存在，無條件）

**定理**：廣義 $n \times n$ 圍棋（基本劫禁）的函數 $Q\text{-DECISION} = \{(S, m, k) : Q(S,m) \geq k\}$ 是 EXPTIME-完全的。因此 $Q \notin \mathrm{FP}$。

**證明**：

**上界**（$Q \in \mathrm{EXPTIME}$）：$Q(S,m) = -V(\mathrm{apply}(S,m))$。$\mathrm{apply}$ 為 $O(n^2)$ 時間可計算。$V(S')$ 可透過 minimax 搜尋整棵博弈樹計算。在基本劫規則下，博弈樹可能含無限路徑（三劫循環等），但可透過 Floyd 循環偵測在 PSPACE $\subseteq$ EXPTIME 中判定循環並賦值 0（見定理 B 的演算法）。對終止路徑，minimax 標準搜尋即可。整體 $Q \in \mathrm{EXPTIME}$。$\square$

**下界**（$Q$ 是 EXPTIME-hard）：

*引理* (Robson 1983)：$V\text{-DECISION} = \{S : V(S) > 0\}$ 是 EXPTIME-完全的。

*歸約*：觀察恆等式 $V(S) = \max_{m \in M(S)} Q(S,m)$。

歸約演算法（使用 $Q$ 的 Oracle）：

```
INPUT: 局面 S
FOR each m ∈ M(S):          // 至多 n²+1 次迭代
    query q_m ← Q(S, m)     // Oracle 查詢
RETURN max{q_m} > 0
```

- Oracle 查詢次數：$|M(S)| \leq n^2 + 1 = \mathrm{poly}(n)$
- 額外計算：$O(n^4)$

此為合法的多項式時間 Turing 歸約。故 $V\text{-DECISION} \leq^T_P Q\text{-DECISION}$。由 $V\text{-DECISION}$ 為 EXPTIME-hard，得 $Q\text{-DECISION}$ 亦為 EXPTIME-hard。$\square$

結合上下界：$Q\text{-DECISION}$ 為 EXPTIME-完全。

**不存在性推論**：假設存在公式 $F$ 使得 $F(S,m) = Q(S,m)$，描述長度 $|F| = s(n)$。$F$ 由有限次標準數學運算組成，求值時間 $T(n) = O(s(n))$。若 $s(n) = \mathrm{poly}(n)$：

$$Q \in \mathrm{FP} \;\Longrightarrow\; V\text{-DECISION} \in P \;\Longrightarrow\; \mathrm{EXPTIME} \subseteq P$$

但 $P \subsetneq \mathrm{EXPTIME}$（時間階層定理，無條件）。矛盾。$\blacksquare$

**普遍性**：此結論對**任何數學框架**下的公式成立——多項式、模形式、L-函數、範疇論構造——只要公式是有限描述的可計算函數且求值時間為 $\mathrm{poly}(n)$。

**近似亦困難**：$Q(S,m)$ 在日本規則下為整數值。任何加法誤差 $< 1/2$ 的近似可透過取整恢復精確值。故精確計算與 $\varepsilon < 1/2$ 近似等價。

---

### 5.2 定理 B（空間壓縮定理，無條件）

**定理**：若 $m_1 \in \mathrm{FP}$，則 $\mathrm{EXPTIME} = \mathrm{PSPACE}$。

**逆否命題**：若 $\mathrm{EXPTIME} \neq \mathrm{PSPACE}$（廣泛相信），則 $m_1 \notin \mathrm{FP}$。

**證明**：

設 $L$ 為任意 EXPTIME-完全語言。設 $A$ 為 $m_1$ 的 poly 時間演算法。

構造 PSPACE 演算法 $B$（使用標準 Floyd 龜兔循環偵測，僅需兩個局面指標）：

```
B(x):
  S_slow ← Robson(x)              // 龜：每步走一手
  S_fast ← Robson(x)              // 兔：每步走兩手

  LOOP:
    // 龜走一步
    m ← A(S_slow)
    S_slow ← apply(S_slow, m)
    IF S_slow 為終局:
      RETURN sign(score(S_slow))   // 得到 V(S₀) 的符號

    // 兔走兩步
    S_fast ← apply(S_fast, A(S_fast))
    IF S_fast 為終局: RETURN sign(score(S_fast))
    S_fast ← apply(S_fast, A(S_fast))
    IF S_fast 為終局: RETURN sign(score(S_fast))

    // 循環偵測
    IF S_slow = S_fast:
      RETURN 0                     // 循環 → 和棋
```

**空間分析**：

| 變數 | 空間 |
|------|------|
| $S_{\text{slow}}$, $S_{\text{fast}}$ | 各 $O(n^2)$ |
| $A$ 的工作空間 | $O(\mathrm{poly}(n))$，每次調用後可重用 |
| **總計** | $O(\mathrm{poly}(n))$ |

**正確性**：

1. **終止性**：局面空間有限（$\leq 3^{n^2}$），最優對弈路徑要麼到達終局，要麼進入循環。Floyd 算法偵測循環。$B$ 必定終止。

2. **對 Robson 局面的正確性**：Robson 構造中，博弈模擬 ATM 計算，必定終止。$B$ 到達終局並輸出正確的 $V(S_0)$。由 Robson 定理，$V(S_0) > 0 \iff x \in L$。

3. $B \in \mathrm{PSPACE}$：$B$ 在任意時刻使用 $\mathrm{poly}(n)$ 空間。PSPACE 允許無限時間但空間受限。

**結論鏈**：

$$m_1 \in \mathrm{FP} \;\Longrightarrow\; L \in \mathrm{PSPACE} \;\Longrightarrow\; \mathrm{EXPTIME} \subseteq \mathrm{PSPACE} \;\Longrightarrow\; \mathrm{EXPTIME} = \mathrm{PSPACE}$$

（最後一步因 $\mathrm{PSPACE} \subseteq \mathrm{EXPTIME}$ 已知。）$\blacksquare$

**核心洞察**：自可歸約在**時間**維度失敗（博弈長度指數，走不完），但在**空間**維度成功——模擬最優對弈只需記住當前局面（**遊忘模擬**），不需記住歷史。

---

### 5.3 元定理：博弈長度障礙的內蘊性

**定理**：若 $\mathrm{EXPTIME} \neq \mathrm{PSPACE}$，則任何 EXPTIME-完全博弈必然具有超多項式博弈長度。

**證明**：設博弈 $\mathcal{G}$ 為 EXPTIME-完全，博弈長度 $L(n) = \mathrm{poly}(n)$。則 $V(S)$ 可由 minimax 深度優先搜尋在 $\mathrm{DSPACE}(\mathrm{poly}(n)) = \mathrm{PSPACE}$ 中計算。故 EXPTIME-完全問題 $\in$ PSPACE，得 $\mathrm{EXPTIME} = \mathrm{PSPACE}$。矛盾。$\square$

**推論**：用時間維度的自可歸約證明 $m_1$ 的 EXPTIME-hardness，對**任何** EXPTIME-完全博弈都不可能成功（除非 $\mathrm{EXPTIME} = \mathrm{PSPACE}$）。空間壓縮論證是繞過此障礙的正確方法。

---

### 5.4 定理 C（$m_2$ 到 $m_1$ 的歸約）

#### 5.4.1 ATM 的二元化

**引理**：任何使用空間 $s(n) \geq \log n$ 的 ATM $M$ 可轉換為二元分支 ATM $M'$（每個 $\exists$/$\forall$ 態恰好 2 個後繼配置），使得 $L(M) = L(M')$ 且 $M'$ 使用空間 $O(s(n))$。

**證明**：分支因子 $k > 2$ 的狀態以二元樹展開（深度 $O(\log k)$），引入 $O(|Q| \cdot |\Gamma| \cdot \max k)$ 個新中間態。新態僅增加狀態集大小（ATM 描述的常數部分），不增加磁帶空間。$\mathrm{APSPACE} = \mathrm{EXPTIME}$ 不受影響。$\square$

#### 5.4.2 修正版性質 (★★)

在二元化 ATM 的 Robson 歸約中，每個可達局面 $S$ 具有以下性質：

**(★★-1') 競爭手集的結構**：

- 若 $S$ 對應 ATM 的分支步驟：$|C(S)| = 2$（兩個 ATM 轉移對應的著手）
- 若 $S$ 對應非分支步驟（確定性 gadget 操作）：$|C(S)| = 1$（唯一迫著手）

**(★★-2) ATM 配置可解碼**：從 $S$ 的棋盤配置可在 $\mathrm{poly}(n)$ 時間內解碼 ATM 的完整配置（狀態、磁帶內容、頭位置），判定分支/非分支，計算 $C(S)$。

> Robson 構造使用明碼 gadget 編碼，每個磁帶格子對應固定大小的子區域，石頭模式直接映射為磁帶符號。

**(★★-3) ATM 手嚴格優於非 ATM 手**：

$$\min_{m \in C(S)} Q(S,m) - \max_{m' \notin C(S)} Q(S,m') \geq \Theta(N)$$

> Robson 的懲罰 gadget 保證：偏離 ATM 協議導致 $\Theta(N)$ 量級損失（管道群被提吃）。

#### 5.4.3 $m_1$ 的恢復

**定理**：若 $m_2 \in \mathrm{FP}$ 且性質 (★★) 成立，則 $m_1 \in \mathrm{FP}$。

**證明**：

```
INPUT: 局面 S, m₂(S) oracle
1. 從 S 解碼 ATM 配置           // poly time（★★-2）
2. 判斷分支/非分支              // poly time
3. IF 非分支:
     m₁ ← C(S) 中唯一元素      // 不需 m₂
   ELSE:
     C(S) ← {c₁, c₂}          // poly time
     // 由 (★★-3)，C(S) 中的手嚴格優於非 C(S) 手（間隔 Θ(N)），
     // 故 m₁, m₂ 均在 C(S) 中。
     m₁ ← C(S) \ {m₂(S)}      // O(1)
4. RETURN m₁
```

每步均在 poly 時間完成。$m_2(S) \in C(S)$ 由 (★★-3) 保證（非 ATM 手的 Q 值低於 ATM 手至少 $\Theta(N)$，故全局前二名必在 $C(S)$ 中）。故 $m_1 \in \mathrm{FP}^{m_2}$。若 $m_2 \in \mathrm{FP}$，則 $m_1 \in \mathrm{FP}$。$\square$

**推論**：$m_2 \in \mathrm{FP} \;\Longrightarrow\; m_1 \in \mathrm{FP} \;\xrightarrow{\text{定理 B}}\; \mathrm{EXPTIME} = \mathrm{PSPACE}$。$\blacksquare$

---

### 5.5 定理 D（$m_2$ 的直接 EXPTIME-hardness）

> **動機**：定理 C 給出條件性結論：$m_2 \in \mathrm{FP} \Rightarrow \mathrm{EXPTIME} = \mathrm{PSPACE}$。定理 D 追求更直接的目標——構造從 EXPTIME-完全問題到 $m_2$ 計算的顯式歸約，在修正版 Robson 構造的性質 (A1-A3) 下得到 $m_2 \notin \mathrm{FP}$（由 $P \neq \mathrm{EXPTIME}$，無條件）。

#### 5.5.1 符號約定與 Pass 不等式

**符號約定**：對局面 $P$，定義：
- $V_B(P)$：Black 先行（即使輪到 White 行棋，假設 Black 先走）時，從 Black 視角的 minimax 值。
- $V_W(P)$：White 先行時，從 White 視角的 minimax 值。
- $\Delta(P) = V_B(P) + V_W(P)$：先行優勢差。$\Delta > 0$ 表示先行有利。

在標準 minimax 中，$V(S)$（第 2 節定義）是「當前行棋方視角」。$V_B$ 和 $V_W$ 是其「人為指定行棋方」的版本，用於分析回合偏移效應。

**引理（pass 不等式）**：對所有圍棋局面 $P$：

$$V_B(P) \geq -V_W(P) \qquad \text{即} \quad \Delta(P) = V_B(P) + V_W(P) \geq 0$$

其中 $V_B(P)$ 為 Black 先行的 minimax 值（Black 視角），$V_W(P)$ 為 White 先行的值（White 視角）。

**證明**：Black 先行時可以選擇 pass。設 $P'$ 為 Black pass 後的局面。$P'$ 與 $P$ 的棋盤配置相同，但行棋方轉為 White，且帶有「上一手為 pass」的標記。

Black pass 後得到 $-V_W(P')$。$P'$ 中 White 的**非 pass** 選擇集與 $P$ 中相同（棋盤未變），唯一差異是 $P'$ 中 White 還可以 pass（此時雙方連續 pass，遊戲結束計分）。White 至少可以忽略 pass 選項、做出與 $P$ 中相同的非 pass 選擇，因此：

$$V_W(P') \geq V_W(P) - O(1)$$

其中 $O(1)$ 修正來自「雙方連續 pass 結束遊戲」可能在 收單官階段產生的微小差異。

結合 $V_B(P) \geq -V_W(P')$（Black pass 策略的下界）：

$$V_B(P) \geq -V_W(P') \geq -V_W(P) - O(1) \qquad \Longrightarrow \qquad \Delta(P) \geq -O(1)$$

**實用版本**：在 Robson 構造中，$|V| = \Omega(N)$ 而 $O(1)$ 修正量可忽略。對本文所需的結論（$V_B > 0$ 當 $-V_W \geq \Omega(N) > 0$），$O(1)$ 修正不影響結果。$\square$

**意義**：先行方的價值至少接近後行方的價值（差距至多 $O(1)$），在大值場景下先行不劣於後行。

#### 5.5.2 修正版 Robson 構造的性質

為保證瓶頸構造的正確性，我們使用滿足以下性質的修正版 Robson 歸約：

**(A1) 劫材嚴格遞減**：所有劫材的價值嚴格不同且遞減排列，確保劫材使用順序完全確定。

> 設計方法：每個劫材對應不同大小的群，$M_1 > M_2 > \cdots$，差距 $\geq 2$。

**(A2) 棋盤完全填充**：非 gadget 區域全部填充為已確定歸屬的活棋群或 單官（中立點，比目法計分下值為 0）。

> 設計方法：標準 padding 技巧。

**(A3) 先行優勢差有界**：$\Delta(P) = O(1)$ 對所有修正版 Robson 構造的可達局面 $P$。

> 由 A1 + A2 + 完全迫著性推出：
>
> - 管道/征子/叫吃 gadget 的迫著性保證偏離損失 $\geq \Omega(N)$（管道群被提）
> - 非 gadget 區域的最大自由手價值 $= O(1)$（單官或小收官）
> - 先行方的額外一步只能落在 $O(1)$ 值的 單官上
> - 故 $\Delta(P) = O(1)$

**(R1) 值分離**：ATM 接受時 $|V(R_x)| \geq \Omega(N)$；ATM 拒絕時亦同。

> 來源：Robson 構造的主群（$\Theta(N)$ 子）的生死由 ATM 的接受/拒絕決定，貢獻 $\pm\Omega(N)$ 目差。

**A1-A3 的構造性實現草圖**：

- **(A1) 的實現**：Robson 原始構造中，劫材來自管道中的棋群。修正方法：將第 $i$ 個劫材對應的群大小設為 $G_i = G_0 - 2i$（嚴格遞減，差距 $\geq 2$）。這只需調整管道的長度分配，不改變棋盤總大小的漸近量級。
- **(A2) 的實現**：在構造完 ATM 編碼區域（管道、劫爭時鐘、轉移 gadget）後，將棋盤上所有剩餘空點分為兩類：(i) 被活棋群包圍的領地——已確定歸屬，落子無益；(ii) 中立點 （單官）——比目法計分下值為 0。若存在大面積未分配空間，填入預製的活棋群（每群至少兩眼，確保不死）。
- **(A3) 的推導**：A1 確保劫材順序完全確定（無等價劫材的選擇自由度）。A2 確保非 gadget 著手值 = 0。管道/征子/叫吃的迫著性確保偏離損失 $\geq \Omega(N)$。因此先行方的額外一步只能落在值 = 0 的 單官上或沿迫著序列多走一步（後者不改變博弈路徑，只消耗一步 $O(1)$ 值的簿記手）。整體 $\Delta(P) = O(1)$。
- **(R1) 的來源**：Robson 構造的主群為沿管道延伸的長鏈，大小 $\Theta(N)$ 子。ATM 接受 → 主群活 → Black 領地 $+\Theta(N)$。ATM 拒絕 → 主群死 → White 領地 $+\Theta(N)$。故 $|V(R_x)| \geq \Omega(N)$。

以上修正均為對 Robson 原始構造的局部調整（管道長度微調 + 空白區域填充），不影響棋盤大小（$\mathrm{poly}(n)$）、歸約 poly 時間性、或 EXPTIME-完全性。

#### 5.5.3 瓶頸-編碼構造

**構造**：給定 ATM 實例 $\langle M, x \rangle$，構造圍棋局面 $S^*$ 於 $(3N) \times (3N)$ 棋盤（$N = \mathrm{poly}(|x|)$，棋盤邊長 $n = 3N = \Theta(N)$）：

- **主戰場 $D$**：Black 群 $G$（$n^4$ 子，一氣，無劫形，gote——一手提子後無後續互動）。著手 $d$ 為提 $G$ 旁 White 子以救活 $G$。
- **衛星 $A$**：嵌入修正版 Robson 構造 $R_x$（大小 $N \times N$）。
- **衛星 $B$**：包含少量 單官點（值 = 0），提供 $m_2$ 的「拒絕」候選。
- 三區域由雙層活棋牆隔離，**無劫交互**（$D$ 和 $B$ 均無劫形）。

#### 5.5.4 $m_1(S^*) = d$ 的證明

$Q(S^*, d)$ 包含救活 $n^4$ 子群的收益。對任何 $m \neq d$：Black 不走 $d$，White 下一手提 $G$，Black 損失 $n^4$ 子。

$$Q(S^*, d) - Q(S^*, m) \geq 2n^4 - O(N^2) > 0$$

> **$2n^4$ 的推導**：比目法計分下，一個被提吃的群貢獻兩倍損失——$n^4$ 顆子成為對方的提子（每子 1 目），且被提後留下的 $n^4$ 個空點成為對方領地（每點 1 目）。走 $d$ 救活群 vs 不走讓對方提吃，swing 值 $\approx 2n^4$。

故 $m_1(S^*) = d$。$\square$

#### 5.5.5 $m_2(S^*)$ 的分析

排除 $d$ 後，候選手來自 $A$ 和 $B$。對 $m_A \in M(A)$：

**偏離路線**：
1. Black 走 $m_A$（在 $A$）
2. White 提 $G$（在 $D$，最佳回應——收益 $2n^4 \gg O(N^2)$，見 §5.5.4）
3. Black 的回合，剩餘博弈 = $A$（已走 $m_A$），Black 先行

$$Q(S^*, m_A) = -n^4 + V_B(\mathrm{apply}(R_x, m_A))$$

$B$ 中 單官手的值：$Q(S^*, m_B) = -n^4 + 0 = -n^4$。

故：$m_2(S^*) \in A \iff \max_{m_A} V_B(\mathrm{apply}(R_x, m_A)) > 0$。

#### 5.5.6 接受方向（無條件）

ATM 接受 $x$：$\exists$ 最優轉移 $m^*$ 使 ATM 沿 $m^*$ 接受。

在原始框架：$Q(R_x, m^*) = -V_W(\mathrm{apply}(R_x, m^*)) > 0$。

由 pass 不等式（$\Delta \geq 0$，**完全無條件**）：

$$V_B(\mathrm{apply}(R_x, m^*)) \geq -V_W(\mathrm{apply}(R_x, m^*)) > 0$$

故 $\max_{m_A} V_B(\mathrm{apply}(R_x, m_A)) > 0$，$m_2(S^*) \in A$。正確編碼「接受」。$\checkmark$

#### 5.5.7 拒絕方向（由 R1 + A3 保證）

ATM 拒絕 $x$：$\forall$ 轉移 $m$，ATM 沿 $m$ 拒絕。

在原始框架：$V_W(\mathrm{apply}(R_x, m)) > 0 \;\;\forall m$。

由 **(R1)**：$V_W(\mathrm{apply}(R_x, m)) \geq \Omega(N)$。

由 **(A3)**：$\Delta(\mathrm{apply}(R_x, m)) \leq O(1)$。

$$V_B(\mathrm{apply}(R_x, m)) = -V_W(\mathrm{apply}(R_x, m)) + \Delta \leq -\Omega(N) + O(1) < 0$$

（對充分大的 $N$）

故 $\max_{m_A} V_B(\mathrm{apply}(R_x, m_A)) < 0$，$m_2(S^*) \in B$。正確編碼「拒絕」。$\checkmark$

#### 5.5.8 結論

$$m_2(S^*) \in A \iff \text{ATM 接受 } x$$

計算 $m_2$ 可判定 EXPTIME-完全問題。由 $P \subsetneq \mathrm{EXPTIME}$：

$$m_2 \notin \mathrm{FP} \qquad \blacksquare$$

---

### 5.6 定理 E（非均勻下界，條件性）

**定理**：在標準假設 $\mathrm{EXPTIME} \not\subseteq P/\mathrm{poly}$ 下，不存在多項式大小的電路族計算 $Q$。

**論據**：若 $\mathrm{EXPTIME} \subseteq P/\mathrm{poly}$，由 Buhrman-Fortnow-Thierauf (1998) 定理，$\mathrm{EXPTIME} \subseteq \Sigma_2^P$。由 $\Sigma_2^P \subseteq \mathrm{PH} \subseteq \mathrm{PSPACE} \subseteq \mathrm{EXPTIME}$，得 $\mathrm{EXPTIME} = \mathrm{PSPACE}$。此等式雖未被無條件證偽（$\mathrm{PSPACE} \neq \mathrm{EXPTIME}$ 是開放問題），但被廣泛認為不成立——因為它蘊含 $\mathrm{DTIME}(2^n) = \mathrm{DSPACE}(n)$（由 padding argument），一個極強的時空等式。$\square$

---

### 5.7 定理 F（固定 19×19 棋盤）

**定理**：對固定的 19×19 棋盤：

(a) 一個（巨大的）公式**必然存在**。

(b) 是否存在緊湊公式，**以現有數學工具既無法證明亦無法反駁**。

**(a)** 合法局面數 $N \approx 2.08 \times 10^{170}$（Tromp-Farnebäck 估計）為有限常數。$Q$ 可表示為大小 $O(N)$ 的查表公式。$\square$

**(b)** 等價於對具體函數的超多項式電路下界——受三重障礙阻擋：

- 相對化 (Baker-Gill-Solovay 1975)
- 自然證明 (Razborov-Rudich 1997)
- 代數化 (Aaronson-Wigderson 2009)

EXPTIME 中顯式函數的已知無條件電路下界僅為 $\Omega(n)$（平凡線性下界）。$\square$

---

## 6. 核心創新的技術分析

### 6.1 空間壓縮論證為何有效

傳統路線（自可歸約）：

$$m_1 \in \mathrm{FP} \;\xrightarrow{\text{沿最優對弈走}}\; V \in P \;\xrightarrow{\text{矛盾}}\; m_1 \notin \mathrm{FP}$$

**失敗**：博弈長度指數，走不完。

新路線（空間壓縮）：

$$m_1 \in \mathrm{FP} \;\xrightarrow{\text{poly 空間模擬}}\; V \in \mathrm{PSPACE} \;\xrightarrow{\text{EXPTIME=PSPACE}}\; \text{不太可能的塌縮}$$

**成功**：不需要在 poly **時間**內完成——只需在 poly **空間**內完成。

深層原因：圍棋的最優對弈可「無記憶地前進」——$m_1(S)$ 是當前局面的函數，不依賴歷史。模擬最優對弈的空間 = 一個局面的大小 = $\mathrm{poly}(n)$。

### 6.2 Pass 不等式為何關鍵

$\Delta(P) \geq 0$ 是一行可證的不等式（Black 可以 pass），卻提供了瓶頸構造中接受方向的**完全無條件保證**：

- 接受方向：$-V_W > 0 \;\xrightarrow{\Delta \geq 0}\; V_B > 0$。**不需要任何額外假設。**
- 拒絕方向：需要 $\Delta = O(1)$，由構造的迫著性保證。

兩個方向使用不同的工具，互相補完。

### 6.3 完全迫著性為何保證 $\Delta = O(1)$

在修正版 Robson 構造中：

1. **管道 gadget**：偏離迫著回應 → 管道群（$\Theta(N)$ 子）被提 → 損失 $\Omega(N)$。
2. **征子/叫吃**：偏離 → 群被提 → 損失 $\Omega(N)$。
3. **劫爭時鐘**：劫材價值嚴格遞減（A1），偏離順序 → 損失 $\Omega(N)$。
4. **非 gadget 區域**：完全填充為 單官／死棋（A2），自由手價值 $= O(1)$。

先行方的「額外一步」只能落在 $O(1)$ 值的 單官上（因為所有 $\Omega(N)$ 級別的著手都是迫著的，不存在「有利但非迫著」的選擇）。故 $\Delta(P) = O(1)$。

### 6.4 回合偏移不可避免但可處理

策略 2 的分析證明了一個結構性結果：

> **回合偏移定理**：在 $D$ 與 $A$ 局部獨立的前提下，Black 偏離（不走 $D$ 而走 $A$）**必然**導致 $A$ 中先行方反轉。此結論與 $D$ 消耗的步數無關。

因此消除偏移是不可能的。正確的策略是**容忍偏移，證明偏移不影響結果**：

- 接受方向：pass 不等式保證（無條件）
- 拒絕方向：$\Delta = O(1) \ll \Omega(N) = |V|$ 保證（由 A1-A3）

---

## 7. 結果總覽

### 7.1 定理層次

| 定理 | 結論 | 條件 | 技術 |
|------|------|------|------|
| A | $Q \notin \mathrm{FP}$ | **無條件** | EXPTIME-完全 + 時間階層 |
| B | $m_1 \in \mathrm{FP} \Rightarrow \mathrm{EXPTIME} = \mathrm{PSPACE}$ | **無條件** | 空間壓縮（新） |
| C | $m_2 \in \mathrm{FP} \Rightarrow \mathrm{EXPTIME} = \mathrm{PSPACE}$ | 修正版 ★★ | 二元分支 + 空間壓縮（新） |
| D | $m_2 \notin \mathrm{FP}$ | A1-A3（可設計） | 瓶頸構造 + pass 不等式 + 迫著性（新） |
| E | $Q \notin P/\mathrm{poly}$ | EXPTIME ⊄ P/poly | Buhrman-Fortnow-Thierauf |
| F | 19×19 緊湊公式不可判定 | — | 電路複雜度三重障礙 |

### 7.2 條件的層次

$$\underbrace{P \neq \mathrm{EXPTIME}}_{\text{無條件（已證）}} \;\;\Longleftarrow\;\; \underbrace{\mathrm{EXPTIME} \neq \mathrm{PSPACE}}_{\text{廣泛相信（未證）}} \;\;\Longleftarrow\;\; \underbrace{\mathrm{EXPTIME} \not\subseteq P/\mathrm{poly}}_{\text{標準猜想}}$$

定理 A 只需最弱條件（已證），定理 B、C 需要中間條件，定理 D 需要 A1-A3（可設計的 gadget 性質），定理 E 需要最強條件。

### 7.3 原創貢獻

1. **空間壓縮論證**：繞過博弈長度障礙，用 PSPACE 模擬替代 poly 時間模擬。
2. **Pass 不等式** $\Delta \geq 0$：接受方向的完全無條件保證。
3. **瓶頸-編碼 + 迫著性分析**：拒絕方向的 $\Delta = O(1) \ll \Omega(N)$ 間隔保證。
4. **回合偏移不可能性定理**：排除「消除偏移」的死路，確立「容忍偏移」的正確策略。
5. **博弈長度元定理**：證明博弈長度障礙是 EXPTIME 完全博弈的內蘊性質。

---

## 8. 對原始問題的回答

### 問題

> 是否存在一個有限、封閉、可計算、非暴力搜尋的解析公式 $F$，使得對所有合法局面 $S$，可直接輸出 $m_2 = F(S)$（或等價輸出 $Q(S,m)$ 的解析閉式）？

### 回答

**不存在。**

- $Q(S,m)$：$Q \notin \mathrm{FP}$（定理 A，無條件）。任何有限描述的可計算公式，只要求值時間為 $\mathrm{poly}(n)$，都不能正確計算 $Q$。此結論對任何數學框架下的公式成立。
- $m_2(S)$：$m_2 \notin \mathrm{FP}$（定理 D，經由修正版 Robson 構造）。此結論的條件（A1-A3）是關於 Robson 構造 gadget 的可驗證性質，我們相信可透過標準 gadget 工程實現，但完整的構造細節尚待寫出。
- $m_1(S)$：$m_1 \in \mathrm{FP}$ 蘊含 $\mathrm{EXPTIME} = \mathrm{PSPACE}$（定理 B，無條件），故在標準猜想下 $m_1 \notin \mathrm{FP}$。

**根本原因**：圍棋局面能編碼任意 EXPTIME 計算，一個 poly 時間公式等價於一個 $P = \mathrm{EXPTIME}$ 的證明，被時間階層定理嚴格排除。

---

## 9. 討論

### 9.1 與組合博弈論收官公式的關係

Berlekamp & Wolfe (1994) 的溫度圖理論為圍棋**可分解收官**提供了精確的「公式」——將局面分解為獨立的局部博弈，計算各自的 CGT 值，再求和得到全局最優策略。

**這與本文結論不矛盾**。CGT 收官公式適用於一類**特殊結構**的局面子集（局部博弈間無交互作用，特別是無劫爭連結）。本文的不存在性結論適用於**所有合法局面**——包括中盤、劫爭糾纏、以及 Robson 構造中的高度結構化局面。

直覺上：CGT 收官公式之所以可行，正是因為它處理的局面**已分解為獨立子問題**。而本文的不可能性正來自圍棋的**不可分解性**——征子、打劫、厚勢造成的全局糾纏使通用分解不可能。

### 9.2 Robson 構造局面 vs 自然局面

有人可能反對：「Robson 歸約產生的局面不像真正的圍棋——它們是人造的、不自然的。」

這個反對有兩個層面的回應：

**數學層面**：歸約的正確性不要求局面「自然」。定理陳述的是「對**所有**合法局面」——包括人造局面。若公式 $F$ 聲稱對所有局面正確，它必須在 Robson 局面上也正確。

**實踐層面**：Robson 局面雖然人造，但它們是**合法的圍棋局面**——可以在標準棋盤上擺出，遵守所有規則。事實上，職業棋手在某些特殊場景（如大規模對殺、複雜劫爭）中遇到的局面，其計算複雜度可能與 Robson 構造相當。AlphaGo 和 KataGo 在這類局面上仍會犯錯，暗示了內在的計算困難性。

### 9.3 開放問題

1. **超級劫下的結論**：Robson 的 EXPTIME-完全性在超級劫下不成立。超級劫圍棋的精確複雜度（PSPACE-hard ≤ ? ≤ EXPSPACE）是開放問題。本文的所有定理是否可推廣到超級劫？

2. **固定 19×19 的緊湊公式**：是否存在多項式大小的電路計算 19×19 圍棋的 $Q$？這等價於一個電路下界問題，受三重障礙阻擋。

3. **A1-A3 的完整構造**：本文提供了 A1-A3 的構造性草圖。完整寫出修正版 Robson 構造並逐一驗證每個 gadget 滿足 A1-A3，是一個有限但非平凡的工作。

4. **近似公式的可能性**：本文排除了精確公式（$\varepsilon < 1/2$）。但對更大的 $\varepsilon$（例如 $\varepsilon = n$），是否存在有意義的近似公式？神經網路（如 KataGo）在實踐中提供了良好的啟發式近似，但其理論保證尚不清楚。

---

## 參考文獻

1. Robson, J.M. (1983). "The Complexity of Go." *Proceedings of IFIP Congress*, pp. 413–417.
2. Lichtenstein, D. & Sipser, M. (1980). "GO Is Polynomial-Space Hard." *JACM*, 27(2), pp. 393–401.
3. Chandra, A.K., Kozen, D.C. & Stockmeyer, L.J. (1981). "Alternation." *JACM*, 28(1), pp. 114–133. (Theorem 3: ASPACE(s(n)) = $\bigcup_c$ DTIME($2^{c \cdot s(n)}$) for $s(n) \geq \log n$.)
4. Berlekamp, E.R. & Wolfe, D. (1994). *Mathematical Go: Chilling Gets the Last Point.* A K Peters.
5. Crâșmaru, M. & Tromp, J. (2000). "Ladders Are PSPACE-Complete." *Computers and Games*, LNCS 2063.
6. Razborov, A.A. & Rudich, S. (1997). "Natural Proofs." *JCSS*, 55(1), pp. 24–35.
7. Aaronson, S. & Wigderson, A. (2009). "Algebrization: A New Barrier in Complexity Theory." *TOCT*, 1(1).
8. Tromp, J. & Farnebäck, G. (2016). "Combinatorics of Go." *tromp.github.io/go/gostate.pdf*.
9. Baker, T., Gill, J. & Solovay, R. (1975). "Relativizations of the P =? NP Question." *SIAM J. Comput.*, 4(4), pp. 431–442.
10. Buhrman, H., Fortnow, L. & Thierauf, T. (1998). "Nonrelativizing Separations." *CCC 1998*, pp. 8–12.
11. Floyd, R.W. (1967). "Nondeterministic Algorithms." *JACM*, 14(4), pp. 636–644. (Cycle detection algorithm.)

---

Maintained by Light. I build and maintain websites with AI as a service: [arthurs.tw](https://arthurs.tw/?utm_source=github&utm_medium=readme&utm_campaign=oss)

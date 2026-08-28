# Regression — GS003-F001 Comparable Population Assembly

> Status: `PERMANENT_REGRESSION_CANDIDATE_ACCEPTED`  
> Source: `GS-003_QUERY_SCOPE_COMPARABLE_POPULATION`  
> Discovery Baseline: `07602b551643ba0bc6d21ff4e3c24bf2e5a0b7e1`  
> Production Rule: `docs/methodology/query-scope-and-population-assembly.md`

---

## 1. Purpose

验证可比口径必须先按本年逐月资格表构造正确 Population，再进入分析发动机。

核心业务规则：

> **本年逐月定资格，同一资格同时约束本年与去年同期；逐月取数后合并，再从头分析。**

可比不是新的 Period State，也不是新的 Bennett / Mix-Rate 算法。

---

## 2. Canonical Reproducer

给定本年逐月可比集合：

```text
Jan: E_01
Feb: E_02
Mar: E_03
...
```

且集合随月份扩大。

某门店：

```text
Jan OUT
Feb OUT
Mar IN
Apr IN
```

则 1-4月可比同比中，该门店只允许进入：

```text
202603 vs 202503
202604 vs 202504
```

不得进入 Jan / Feb，也不得因为 2025-03 当时自己的 comparable_flag=0 而再次排除 202503。

---

## 3. Expected Algorithm

对每个本年月份 `t`：

```text
E_t = 本年 t 月 comparable_flag=1 的门店集合
Current_t = Current(t) ∩ E_t
Base_t    = Base(t-12m) ∩ E_t
```

然后：

```text
APPEND all monthly Current/Base pairs
→ new Canonical Analysis Input
→ rerun the full Analysis Engine
```

---

## 4. Forbidden Implementations

### A. Base-year flag double filtering

禁止：

```text
Current comparable = YES
AND Base-year comparable = YES
```

去年同期自己的 comparable_flag 不参与本次本年可比资格判断。

### B. Period-end snapshot backfill

禁止：

```text
拿期末月份可比店名单
→ 回刷整个累计期间
```

### C. Current-only filtering

禁止：

```text
Current = 可比集合
Base = 全量集合
```

### D. Post-calculation filtering

禁止：

```text
先在全量 Parent 计算 Mix / Rate / Materiality / Offset
→ 再从结果表筛 comparable
```

必须先构造 Population，再重算 Contextual Result。

---

## 5. GS-003 Discovery Record

GS-003 设计的本年可比门店数：

```text
Jan 130
Feb 140
Mar 150
Apr 160
May 170
Jun 180
Jul 185
```

同一合成数据下：

```text
All-store YTD GMV YoY ≈ -0.66%
Correct Comparable YTD GMV YoY ≈ +3.70%
```

错误实现：

```text
Require Base-year Comparable Flag ≈ +2.83%
July Snapshot Backfill           ≈ +1.38%
Current-only Filter              ≈ -19.15%
```

说明 Population Assembly 错误可以显著扭曲甚至反转经营判断，即使后续数学公式全部正确。

---

## 6. Failure Conditions

任一满足即 FAIL：

```text
A. 使用去年同期自己的 comparable_flag 二次筛选
B. 用期末可比名单回刷整个累计期
C. Current / Base 使用不同 Population
D. 先计算 Parent-dependent Result 再筛 comparable
E. comparable_flag 被错误塞入 Period State / Transition State Machine
F. 可比 Scope 改变后复用原 Parent 的 Materiality / Mix / Offset
```

---

## 7. Invariants

```text
Eligibility Before Context = PASS
Current/Base Monthly Population Pair = PASS
Query Context Recalculation = PASS
Derived Ratio Recalculation = PASS
No New Period State = PASS
Math Kernel Unchanged = PASS
```

该 Regression 防止：

> **进料集合已经错了，但后面的公式因为仍然闭合而给出“非常可信的错误答案”。**

# Regression — GS003-F002 Arbitrary Comparison Window Recalculation

> Status: `PERMANENT_REGRESSION_CANDIDATE_ACCEPTED`  
> Source: `GS-003_QUERY_SCOPE_COMPARABLE_POPULATION`  
> Discovery Baseline: `07602b551643ba0bc6d21ff4e3c24bf2e5a0b7e1`  
> Production Rule: `docs/methodology/query-scope-and-population-assembly.md`

---

## 1. Purpose

验证：

> **YTD 不是特殊算法；任何跨月 Comparison Window 都必须先聚合底层金额，再重新计算 Derived / Contextual Result。**

典型真实问题：

```text
5-6月
Q2
618周期
双11周期
最近3个月
1-7月YTD
```

---

## 2. Canonical Rule

先定义：

```text
Current Window = {t1, t2, ..., tn}
Base Window    = 对应比较期 {b1, b2, ..., bn}
```

然后在目标 Query Scope 内：

```text
Current Amounts = Σ Current Window 底层金额
Base Amounts    = Σ Base Window 底层金额
```

再重新计算：

```text
Rate
GMV YoY
Parent Bennett
Parent Rate Bridge
Continuing Mix / Rate
Offset
Individual Materiality
Collective Materiality
Attention
```

---

## 3. Forbidden Roll-up

禁止把月度已计算结果直接相加或平均，冒充窗口结果。

包括：

```text
Window Rate Change != SUM(月度 Rate Change)
Window Mix         != SUM(月度 Mix)
Window Rate Effect != SUM(月度 Rate Effect)
Window Materiality != SUM(月度 Materiality)
Window Offset      != AVG / SUM(月度 Offset)
```

除非某个 Measure 的正式数学类型明确允许跨时间 SUM。

---

## 4. Minimal Example

例如：

```text
Month A: GMV 100, GP 10, Rate 10%
Month B: GMV 900, GP 135, Rate 15%
```

窗口总体率：

```text
(10 + 135) / (100 + 900) = 14.5%
```

不是：

```text
10% + 15%
```

也不是无权重平均：

```text
(10% + 15%) / 2
```

---

## 5. GS-003 Discovery Record

GS-003 中：

```text
May Rate YoY ≈ -0.362pp
Jun Rate YoY ≈ -0.591pp
```

错误直接相加：

```text
≈ -0.954pp
```

正确将 5-6月 Amount / Numerator / Denominator 合并后重算：

```text
≈ -0.498pp
```

因此一个很常见的“小范围合并月份分析”，如果沿用月度结果直接 Roll-up，会产生明显错误。

---

## 6. Comparable Window Interaction

若 Query 同时要求 Comparable：

```text
每个月先按本年当月 E_t 构造 Current/Base Pair
→ APPEND 全部月度 Pair
→ 再形成整个 Comparison Window
→ 从头计算
```

因此：

```text
5-6月可比
!=
6月可比名单回刷5-6月
```

也不能先分别算5月、6月可比 Mix/Rate 后再相加。

---

## 7. Failure Conditions

任一满足即 FAIL：

```text
A. SUM 月度 Rate Change 作为 Window Rate Change
B. SUM 月度 Mix / Rate Effect 作为 Window Mix / Rate
C. SUM / AVG 月度 Materiality 作为 Window Materiality
D. AVG 月度 Offset 作为 Window Offset
E. 只支持 YTD 重算，但对其他跨月 Window 使用旧结果 Roll-up
F. Comparable Window 未先完成逐月 Population Pair
```

---

## 8. Production Interpretation

时间统一建模为：

```text
Comparison Window
```

其中：

```text
单月 = 1个月窗口
YTD = 从年初到当前月的窗口
5-6月 = 2个月窗口
618 = 业务定义日期/月份窗口
```

原则：

> **时间范围改变就是 Context 改变；Contextual Result 必须重算。**

该规则不修改 Bennett / Mix-Rate 数学公式，只把既有 YTD 重算原则推广为通用时间合同。

# Query Scope & Population Assembly：查询作用域、可比总体与时间窗口

> **Bundled Runtime Snapshot**：本文件是可安装 Skill 内置的 Scope / Population / Window 合同快照，来源于 `docs/methodology/query-scope-and-population-assembly.md`。  
> 适用范围：单店损益 A 阶段所有 Query-scoped Parent。

---

## 1. 核心结论

真实经营问题通常不是先给定一个固定 Parent，再问为什么变化，而是先通过问题定义本次分析集合。

系统必须先回答：

```text
看谁？
看哪个时间范围？
使用什么业务口径？
Current 与 Base 怎样配对？
```

正式原则：

> **Scope / Population First, Contextual Calculation Second.**

> **先构造本次合法分析总体，再启动 Parent-dependent 计算。**

---

## 2. Query-Scoped Parent

Parent 不是固定为“全部门店 / 整个分部”。用户问题可以动态定义 Parent，例如：

```text
store_type = 超体
channel = 地采
division = 某分部
comparable = YES
time_window = 202605-202606
```

筛选后的事实集合即为本次 Query Scope。

Scope 变化后，以下 Contextual Result 必须重新计算：

```text
Parent Rate
Parent Bennett
Continuing Mix / Rate
Contribution
Offset
Individual Materiality
Collective Materiality
Structured Long-tail
Attention Rank
```

禁止把在更大 Parent 下计算的这些结果直接筛选后复用。

---

## 3. 可比门店不是新的 Period State

上游提供：

```text
门店 × 月份 × comparable_flag
```

分析系统不在本层重新推断可比资格规则；该业务规则由上游状态表负责。

`comparable_flag` 只决定本次分析 Population 是否纳入该门店×月份。

因此 `STANDARD / ABSENT / NEGATIVE_GMV / ...` 描述数学状态，而 `comparable_flag` 描述是否进入当前可比分析总体。二者不得混成同一个 State Machine。

---

## 4. 可比口径正式逐月 Pair 规则

可比资格以**本年当月**状态为准，而不是去年同期自己的可比状态。

对本年月份 `t`：

```text
E_t = {store | comparable_flag(current_year, t, store) = 1}
Current_t = 本年 t 月、门店属于 E_t 的事实
Base_t    = 去年同期 t 月、门店属于同一个 E_t 的事实
```

对所有月份遍历后：

```text
APPEND all monthly pairs
→ Comparable Canonical Analysis Input
→ 从头运行完整 Analysis Engine
```

一句话规则：

> **本年逐月定资格，同一资格同时约束本年与去年同期；逐月取数后合并，再从头分析。**

---

## 5. 可比口径明确禁止

禁止：

1. 再次使用去年同期自己的 comparable_flag 二次筛选；
2. 使用期末可比名单回刷整个累计期；
3. 只筛 Current、不筛 Base；
4. 先跑全量 Contextual Result 再筛 Comparable。

两期 Population 必须按同一业务规则配对。

---

## 6. Comparison Window 与时间粒度

统一定义：

> **Comparison Window（比较时间窗口）**

当前 V1 Canonical Input：

```text
period = YYYYMM
```

正式规则：

> **Comparison Window resolution MUST NOT be finer than the Canonical Input time grain.**

当前原生支持：

```text
单月
5-6月
Q2
1-7月 YTD
最近3个月
其他由完整月份组成的合法月份集合
```

“618”“双11”等业务标签只有在上游输入或版本化 Window Registry 能够把它完整且唯一表达为当前 Canonical Grain 时才支持。

如果用户实际要问 `6/1-6/18`，但输入只有 `YYYYMM` 月度事实，则当前 V1 不支持。

禁止通过以下方式伪造 Sub-month Window：

```text
平均分摊月度金额
按天数比例拆分
AI 推断月内销售曲线
隐式假设某些日期等价于整月
```

> **注册一个 Window 名称不会创造输入中不存在的更细粒度事实。**

---

## 7. Window Recalculation

对合法 Comparison Window：

```text
先定义 Current Window 与对应 Base Window
→ 在窗口内聚合底层 Amount / Numerator / Denominator
→ 重建该 Window 的分析事实集合
→ 从头计算 Contextual Result
```

因此：

```text
Window Rate ≠ SUM(月度 Rate Change)
Window Mix ≠ SUM(月度 Mix)
Window Rate Effect ≠ SUM(月度 Rate Effect)
Window Materiality ≠ SUM(月度 Materiality)
Window Offset ≠ SUM / AVG(月度 Offset)
```

合法多月 Window 必须聚合底层 Amount / Denominator 后重新计算 Contextual Result。

---

## 8. 可比 × Comparison Window

可比 Window 必须先逐月完成 Population Pair，再合并。

例如 5-6 月可比：

```text
202605:
E_05 → 202605 + 202505

202606:
E_06 → 202606 + 202506

↓ APPEND
5-6月 Comparable Fact Set
↓
重新计算 GMV / GP / Rate / Bennett / Mix-Rate / Materiality / Offset
```

因此：

> **5-6月可比 ≠ 6月可比名单的5-6月累计。**

---

## 9. 与现有算法的关系

本规则不修改 Symmetric Bennett、Parent Rate Bridge、Continuing Mix / Rate、Entry / Exit、Non-standard、Time Reversal、Materiality、Collective Materiality、Offset。

它只明确发动机之前的输入总体与时间窗口构造。

正式架构：

```text
User Query
→ Query Parser
→ Scope / Population Resolver
→ Comparison Window Grain Validation
→ Comparison Window Pairing
→ Comparable Monthly Mask（若适用）
→ Canonical Analysis Input for this Query
→ Existing Analysis Engine
```

> **不要为超体、地采、可比、5-6月分别开发四套分析算法；它们只是不同 Scope / Population / Window 定义。**

---

## 10. Production Invariants

### Population Symmetry by Business Rule

若业务定义要求 Current 月资格集合 `E_t` 同时约束 Current 与 Base，则两期必须使用同一 `E_t`。

### Eligibility Before Context

任何 Eligibility / Scope Filter 必须发生在 Parent-dependent Calculation 之前。

### Window Grain Compatibility

Comparison Window 最细 resolution 不得细于 Canonical Input Grain。

### Window Recalculation Required

合法跨月 Window 必须基于窗口底层金额重新构造并计算 Derived / Contextual Result。

### Query Context Recalculation Required

Parent / Scope 改变时，Materiality、Offset、Mix/Rate 等 Contextual Result 必须重算。

---

## 11. 总原则

> **先决定哪些事实属于这道题，再决定这些事实为什么变化。**

> **可比不是新的分析算法，而是一个逐月变化、由本年状态表驱动的 Population Filter。**

> **YTD 不是特殊数学世界，只是 Comparison Window 的一个常见实例。**

> **Comparison Window 不能比输入事实更细；业务标签不能替代不存在的数据。**

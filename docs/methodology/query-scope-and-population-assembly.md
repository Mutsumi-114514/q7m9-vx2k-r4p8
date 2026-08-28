# Query Scope & Population Assembly：查询作用域、可比总体与时间窗口

> 状态：`CURRENT SUPPORTING CONTRACT`  
> 生效日期：2026-08-29  
> 来源：GS-003 Query Scope & Comparable Population Golden Test。  
> 适用范围：单店损益 A 阶段所有 Query-scoped Parent，包括门店类型、渠道、组织、可比口径与任意时间窗口。  
> 上游统一入口：`docs/methodology/production-system-contract.md`。

---

## 1. 核心结论

真实经营问题通常不是先给定一个固定 Parent，再问为什么变化，而是先通过问题定义本次分析集合。

例如：

```text
这个月超体业绩为什么下滑？
这个月地采毛利率为什么下降？
5-6月整体毛利为什么下降？
1-7月可比门店表现怎么样？
```

系统必须先回答：

```text
看谁？
看哪个时间范围？
使用什么业务口径？
Current 与 Base 怎样配对？
```

然后才进入原有 Analysis Engine。

正式原则：

> **Scope / Population First, Contextual Calculation Second.**  
> **先构造本次合法分析总体，再启动 Parent-dependent 计算。**

---

## 2. Query-Scoped Parent

Parent 不是固定为“全部门店 / 整个分部”。

用户问题可以动态定义 Parent，例如：

```text
store_type = 超体
channel = 地采
division = 某分部
comparable = YES
time_window = 202605-202606
```

筛选后的事实集合即为本次 Query Scope。

该 Scope 变化后，以下 Contextual Result 必须重新计算：

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

GS-003 中同一家目标超体：

```text
全部门店 Parent 下 Rate Materiality ≈ 0.0398pp
超体 Parent 下 Rate Materiality ≈ 0.4914pp
```

说明重大性是当前 Query Context 的属性，不是门店自身的永久标签。

---

## 3. 可比门店不是新的 Period State

当前业务上游提供：

```text
门店 × 月份 × comparable_flag
```

分析系统不在本层重新推断“开业后第14个月是否进入可比”；该业务规则由上游状态表负责。

`comparable_flag` 只决定本次分析 Population 是否纳入该门店×月份。

因此：

```text
STANDARD / ABSENT / NEGATIVE_GMV / ...
```

仍然描述数据与业务原子的数学状态；

```text
comparable_flag
```

描述该门店×月份是否进入当前可比分析总体。

两者不得混成同一个 State Machine。

---

## 4. 可比口径的正式逐月 Pair 规则

可比资格以**本年当月**状态为准，而不是去年同期自己的可比状态。

对本年月份 `t`：

```text
E_t = {store | comparable_flag(current_year, t, store) = 1}
```

然后同时构造：

```text
Current_t = 本年 t 月、门店属于 E_t 的事实
Base_t    = 去年同期 t 月、门店属于同一个 E_t 的事实
```

对所有月份遍历：

```text
(Current_1, Base_1)
(Current_2, Base_2)
...
(Current_n, Base_n)
```

最后：

```text
APPEND all monthly pairs
→ 形成新的 Comparable Canonical Analysis Input
→ 从头运行完整 Analysis Engine
```

一句话规则：

> **本年逐月定资格，同一资格同时约束本年与去年同期；逐月取数后合并，再从头分析。**

---

## 5. 可比口径明确禁止的实现

### 5.1 禁止再次使用去年同期自己的 comparable_flag

错误：

```text
current comparable = YES
AND
base-year comparable = YES
```

这会改变业务定义，把本年当前可比总体再次缩小。

GS-003 中：

```text
正确 1-7月可比 GMV同比 ≈ +3.70%
错误“双边都要求可比” ≈ +2.83%
```

### 5.2 禁止期末名单回刷整个累计期

错误：

```text
使用 7月可比门店集合
→ 回刷 1-7月全部月份
```

因为真实可比集合按月变化。

GS-003 中：

```text
正确 ≈ +3.70%
7月名单回刷全年 ≈ +1.38%
```

### 5.3 禁止只筛 Current、不筛 Base

错误：

```text
Current = 本年可比门店
Base = 去年全量门店
```

两期 Population 不一致。

GS-003 中该错误把：

```text
正确 +3.70%
```

错误计算为：

```text
-19.15%
```

足以直接反转经营判断。

### 5.4 禁止先跑全量 Contextual Result 再筛 Comparable

必须先形成新的可比事实集，再从头重算 Parent-dependent Result。

---

## 6. 任意 Comparison Window

时间 Scope 不应只建模为：

```text
单月
YTD
```

统一定义为：

> **Comparison Window（比较时间窗口）**

例如：

```text
单月
5-6月
Q2
1-7月 YTD
618周期
双11周期
最近3个月
其他业务活动期
```

正式规则：

```text
先定义 Current Window 与对应 Base Window
→ 在窗口内聚合底层 Amount / Numerator / Denominator
→ 重建该 Window 的分析事实集合
→ 从头计算 Contextual Result
```

因此：

```text
Window Rate ≠ SUM(月度 Rate Change)
Window Mix  ≠ SUM(月度 Mix)
Window Rate Effect ≠ SUM(月度 Rate Effect)
```

除非某个结果本身被明确证明在该语义下可跨月相加。

GS-003 中：

```text
5月 Rate同比 ≈ -0.362pp
6月 Rate同比 ≈ -0.591pp
错误直接相加 ≈ -0.954pp
正确 5-6月窗口重算 ≈ -0.498pp
```

---

## 7. 可比 × 任意时间窗口

可比 Window 必须先逐月完成 Population Pair，再合并。

例如 5-6月可比：

```text
202605：
E_05 → 202605 + 202505

202606：
E_06 → 202606 + 202506

↓ APPEND

5-6月 Comparable Fact Set
↓
重新计算 GMV / GP / Rate / Bennett / Mix-Rate / Materiality / Offset
```

因此：

> **5-6月可比 ≠ 6月可比名单的5-6月累计。**

---

## 8. 与现有算法的关系

本规则不修改：

```text
Symmetric Bennett
Parent Rate Bridge
Continuing Mix / Rate
Entry / Exit
Non-standard
Time Reversal
Materiality
Collective Materiality
Offset
```

它只明确发动机之前的输入总体构造。

正式架构：

```text
User Query
→ Query Parser
→ Scope / Population Resolver
→ Comparison Window Pairing
→ Comparable Monthly Mask（若适用）
→ Canonical Analysis Input for this Query
→ Existing Analysis Engine
```

原则：

> **不要为超体、地采、可比、5-6月分别开发四套分析算法；它们只是四种 Scope / Population 定义。**

---

## 9. 新增 Production Invariants

### Population Symmetry by Business Rule

若业务定义要求 Current 月的资格集合 `E_t` 同时约束 Current 与 Base，则两期必须使用同一 `E_t`。

### Eligibility Before Context

任何 Eligibility / Scope Filter 必须发生在 Parent-dependent Calculation 之前。

### Window Recalculation Required

任意跨月 Comparison Window 必须基于窗口底层金额重新构造并计算 Derived / Contextual Result。

### Query Context Recalculation Required

Parent / Scope 改变时，Materiality、Offset、Mix/Rate 等 Contextual Result 必须重算。

---

## 10. 当前总原则

> **先决定哪些事实属于这道题，再决定这些事实为什么变化。**

> **可比不是新的分析算法，而是一个逐月变化、由本年状态表驱动的 Population Filter。**

> **YTD 不是特殊数学世界，只是 Comparison Window 的一个常见实例。**

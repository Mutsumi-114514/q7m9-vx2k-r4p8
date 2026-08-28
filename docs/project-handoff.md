# 经营分析 AI Skills 项目交接文档

> 文档状态：`CURRENT PROJECT HANDOFF`  
> 基线日期：2026-08-29  
> 当前阶段：A 阶段数学主干稳定；完成 GS-001 / GS-002 / GS-003、Repository Integration Audit 与 System Interaction Side-effect Test；当前 Production Contract 为 V0.3。  
> 目的：让新对话 / 新 AI 快速恢复当前正确状态，不被历史评审或已被后续规则覆盖的旧执行链带偏。

---

## 0. 新对话读取顺序

1. `README.md`
2. `docs/methodology/production-system-contract.md` — **当前唯一统一 Production Contract**
3. `docs/data-contracts/store-pnl-data-contract.md`
4. `docs/methodology/query-scope-and-population-assembly.md` — GS-003 Scope / 可比 / Time Window
5. `docs/methodology/production-shadow-decomposition.md`
6. `docs/methodology/materiality-gate.md` — GS-001
7. `docs/methodology/collective-materiality.md` — GS-002
8. `docs/testing/system-interaction-side-effect-test.md`
9. `docs/testing/test-sample-specification.md`
10. `docs/testing/golden-reality-profile-east-china-v1.md`
11. `docs/testing/golden-reality-profile-east-china-v1-validation.md`
12. `docs/methodology/multi-agent-review-protocol.md`

Round 2 / Round 3 是历史 Review Evidence，不是 Implementation Source。

若支持文档与 `production-system-contract.md` 冲突：

> **以 Production System Contract + Data Contract 为准。**

---

## 1. 项目目标

```text
定义问题
→ 定义 Query Scope / Population / Comparison Window
→ 验证数据
→ 穷举计算
→ WHAT / WHERE
→ Mathematical WHY
→ Materiality
→ Attention
→ Hypothesis
→ Evidence
→ Action
```

总纲：

> **先把数据做对，再把问题找准；先用数据解释到极限，再去现实世界找证据；简单方法优先，复杂方法必须证明增量价值；AI 负责穷尽执行，人负责定义、质疑和决策。**

> **Exhaustive Calculation, Selective Attention。**

---

## 2. 当前 MVP 与 Query Scope

当前指标主线：

```text
GMV → 综合毛利额 → 综合毛利率
```

Atomic Grain：

```text
时间 × 门店 × 渠道
```

门店类型、分部、大区属于 Attribute / Hierarchy；当前原子渠道为地采、集采、万家、星选，`大集采=集采+万家` 是 Derived View。

真实问题可以动态定义 Parent，例如：

```text
本月超体
本月地采
5-6月整体
1-7月可比门店
某分部 × 某渠道
```

Scope 改变后 Parent Rate、Mix/Rate、Offset、Materiality、Collective Materiality 等 Contextual Result 必须重算。

---

## 3. Data Contract 与 Period State

默认：

```text
综合毛利率 = 综合毛利 / 不含仅双记GMV
不含仅双记GMV = 常规GMV - 仅双记GMV
```

Period State：

```text
ABSENT
STANDARD
NET_ZERO_PRESENT
ZERO_GMV_NONZERO_GP
NEGATIVE_GMV
INVALID_OR_MISSING
OTHER_NONSTANDARD
```

关键纪律：

- Full Outer Join 后先保存 Presence；
- `ABSENT ≠ NET_ZERO_PRESENT`；
- GMV=0 时 Rate=N/A；
- 不自动去重；
- 官方 Control Total 优先；
- 可比资格不是新的 Period State。

---

## 4. 当前数学主干

### GMV

```text
ΔGMV_i = G1_i - G0_i
```

### Continuing Standard 毛利额

```text
Scale = (G1-G0) × (r0+r1)/2
Rate  = (r1-r0) × (G0+G1)/2
Scale + Rate = ΔGP
```

### 原子统一闭合

```text
atomic_gp_effect_total = current_gp - base_gp
Σ atomic_gp_effect_total = Δ Parent GP
```

### Parent Rate

```text
Total Parent
↓ Non-standard
Standard Parent
↓ Exit / Entry
Continuing Standard Parent
↓ Mix / Rate
```

共同存量存在：

```text
Δ Parent Rate = Nonstandard + Exit + Mix + Rate + Entry
```

共同存量为空则走 Membership Replacement。

Time Reversal：同名组件映射反号，Entry ↔ Exit 后反号。

---

## 5. GS-001：Materiality Gate

GS-001 证明：数学闭合不等于 Decision Precision。

正式原则：

> **Abnormality ≠ Materiality。**

Rate 重大性：

```text
Impact_i = |ΔR - ΔR(-i)|
```

Tiny Denominator 不修改原始值、不改 Period State、不封顶 Rate，只进入 Warning / Attention。

永久 Regression：

`docs/testing/regressions/gs001-f001-extreme-rate-low-materiality.md`

---

## 6. GS-002：Collective Materiality

正式原则：

> **Individual Insignificance ≠ Collective Insignificance。**

Materiality 必须在当前 Group / View / Parent Context 下重算，不能把 Atomic Materiality SUM / AVG 成 Group Materiality。

永久 Regression：

`docs/testing/regressions/gs002-f001-collective-materiality.md`

---

## 7. System Interaction / Side-effect Test

组合 Materiality、Collective Materiality、Offset、Long-tail、多 View 后：

- 数学主干仍稳定；
- GS-001 / GS-002 修复未回归；
- 发现一个新的系统风险：同一底层现象可能从多个 View 被重复升级为主要 Finding。

当前安全边界：

> **Projection Is Evidence, Not Additional Effect。**

跨 View Materiality 不相加；多个 View 命中不自动声明多个独立 Driver；关系不明时 `OVERLAP_UNKNOWN`。

`Finding Consolidation` 尚未完成 Human Adjudication / Production Freeze。

---

## 8. GS-003：Query Scope / Comparable / Time Window

GS-003 第一次按真实提问方式测试动态 Parent，而不是固定最大 Parent。

### 8.1 GS003-F001 — Comparable Population Assembly（ACCEPT）

上游提供：

```text
门店 × 月份 × comparable_flag
```

正式规则：

```text
对每个本年月份 t：
E_t = 本年 t 月可比门店集合
Current_t = 本年 t 月 ∩ E_t
Base_t    = 去年同期 ∩ 同一个 E_t

逐月 Pair
→ APPEND
→ 新 Canonical Analysis Input
→ 从头运行完整 Analysis Engine
```

一句话：

> **本年逐月定资格，同一资格同时约束本年与去年同期；逐月取数后合并，再从头分析。**

禁止：

- 去年同期自己的 comparable_flag 再筛一次；
- 期末可比名单回刷整个累计期；
- Current 筛可比而 Base 仍全量；
- 先算全量 Contextual Result 再筛 Comparable。

GS-003 实证：正确 1-7月可比 GMV同比约 `+3.70%`；错误 Current-only Filter 可变成约 `-19.15%`，足以反转结论。

永久 Regression：

`docs/testing/regressions/gs003-f001-comparable-population-assembly.md`

### 8.2 GS003-F002 — Arbitrary Comparison Window（ACCEPT）

YTD 不再视为特殊数学路径，而统一成：

```text
Comparison Window
```

包括单月、5-6月、Q2、YTD、618、双11、最近N个月等。

跨月必须：

```text
先聚合底层 Amount / Numerator / Denominator
→ 重建 Window
→ 重算 Rate / Mix-Rate / Offset / Materiality
```

GS-003 中：

```text
5月 Rate同比 ≈ -0.362pp
6月 Rate同比 ≈ -0.591pp
错误直接相加 ≈ -0.954pp
正确 5-6月窗口重算 ≈ -0.498pp
```

永久 Regression：

`docs/testing/regressions/gs003-f002-arbitrary-comparison-window.md`

---

## 9. Repository Integration Audit / Reality Profile

Repository Integration Audit 已修复 Source of Truth 分裂，并新增统一 Production Contract 与 Cross-View Guard。

Golden Reality Profile：约 11 分部、218 门店、4 原子渠道、门店规模偏斜、渠道异质性高、内部 Offset 常见。

两个已知画像冲突由 Validation Note 管理：

1. 店型数量合计 220 vs store_count 218；
2. 186/218 四渠道齐全 vs 星选覆盖约75%。

Generator 必须区分 Hard Constraint / Soft Reference。

---

## 10. 测试治理

```text
L1 Canonical / Boundary / Regression
L2 Scenario / Golden
L3 Property / Adversarial Random
L4 Multi-Agent Adversarial Review
```

Golden：

```text
Scenario Truth
→ Generator
→ Synthetic Raw Data
→ Independent Oracle
→ Expected Result
→ Black-box Acceptance
```

ACCEPT 反例永久进入 Regression。

当前新增永久 Regression：

```text
GS003-F001 Comparable Population Assembly
GS003-F002 Arbitrary Comparison Window Recalculation
```

---

## 11. 当前未冻结

```text
Materiality 固定阈值
Attention Top N
Finding Consolidation / 跨 View 去重算法
Collective Cohort 最终优先级策略
Interaction Shadow 是否长期保留
```

这些不得提前写死为 Production 事实。

---

## 12. 当前真正的下一步

数学主干目前没有新证据要求推翻。

优先继续验证：

1. `Finding Consolidation` 是否能用最小规则解决 Cross-View Finding Multiplication；
2. GS-003 新增的 Population / Comparison Window 合同是否与 Entry/Exit、Non-standard、Collective Materiality 等边界组合后仍稳定；
3. 后续再继续原计划的 Mix-Rate Paradox、Entry/Exit & Nonstandard、High Offset & YTD/Window 等 Golden。

---

## 13. 新对话一句话启动指令

```text
请接手 operating-analysis-skills 项目。
先读取 README.md、docs/methodology/production-system-contract.md、docs/project-handoff.md。
当前 Production Contract = V0.3；已完成 GS-001 / GS-002 / GS-003、Repository Integration Audit 和 System Interaction Test。
稳定数学公式不要无证据推翻；当前已正式冻结 Query Scope / Comparable Population / Arbitrary Comparison Window，Finding Consolidation 仍待后续裁决与测试。
```

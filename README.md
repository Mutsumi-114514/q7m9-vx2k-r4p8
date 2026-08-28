# Operating Analysis Skills

一套面向经营分析场景的 AI Skills 方法论与工程化实践。

目标不是让 AI 生成一篇“看起来合理”的经营分析，而是把分析过程拆成可验证、可复用、可审计、可测试的执行系统：

```text
定义问题
→ 定义 Query Scope / Population / Time Window
→ 验证数据
→ 穷举计算
→ 定位差异
→ 数学归因
→ 重大性判断
→ 注意力筛选
→ Hypothesis
→ Evidence
→ Action
```

核心原则：

> **先把数据做对，再把问题找准；先用数据解释到极限，再去现实世界找证据；简单方法优先，复杂方法必须证明增量价值；AI 负责穷尽执行，人负责定义、质疑和决策。**

> **Exhaustive Calculation, Selective Attention（计算穷举，注意力筛选）。**

---

## 当前阶段

截至 2026-08-29：

- A 阶段数学主干稳定：Symmetric Bennett、Parent Rate Bridge、Entry / Exit、Non-standard、Full Replacement、Time Reversal、Roll-up；
- 完成 Data Contract / Execution Semantics / Test Sample Specification / Golden Reality Profile V1；
- `GS-001 NORMAL_BASELINE`：修复 Extreme Rate × Low Materiality 解释污染；
- `GS-002 SIZE_STRUCTURAL_MIGRATION`：修复 Individual Insignificance ≠ Collective Insignificance；
- 完成 Repository Integration Audit，并收口统一 Production System Contract；
- 完成 System Interaction / Side-effect Test：数学主干稳定，但发现 Cross-View Finding Multiplication，当前仍待 Finding Consolidation 裁决；
- `GS-003 QUERY_SCOPE_COMPARABLE_POPULATION`：正式接受两项新规则：
  - `GS003-F001` 本年逐月 Comparable Population Assembly；
  - `GS003-F002` Arbitrary Comparison Window Recalculation；
- 当前 Canonical Production Contract 已升级至 V0.3。

---

## 当前唯一运行入口

### Canonical Production Contract

[Production System Contract V0.3](docs/methodology/production-system-contract.md)

运行期实现、测试和 AI 解释优先以该文件为当前统一合同。

### 支撑合同 / 证据

- [Store P&L Data Contract](docs/data-contracts/store-pnl-data-contract.md)
- [Query Scope & Population Assembly](docs/methodology/query-scope-and-population-assembly.md)
- [Production / Shadow 数学分解基线](docs/methodology/production-shadow-decomposition.md)
- [Materiality Gate — GS-001](docs/methodology/materiality-gate.md)
- [Collective Materiality — GS-002](docs/methodology/collective-materiality.md)
- [Impact Ledger 与聚合约束](docs/methodology/impact-ledger-and-rollup.md)
- [Roll-up Engine](docs/methodology/roll-up-engine.md)
- [Skill Execution Architecture](docs/methodology/skill-execution-architecture.md)
- [Test Sample Specification](docs/testing/test-sample-specification.md)
- [System Interaction Side-effect Test](docs/testing/system-interaction-side-effect-test.md)
- [Multi-Agent Review Protocol](docs/methodology/multi-agent-review-protocol.md)
- [Golden Reality Profile V1](docs/testing/golden-reality-profile-east-china-v1.md)
- [Reality Profile Validation Note](docs/testing/golden-reality-profile-east-china-v1-validation.md)
- [Project Handoff](docs/project-handoff.md)

历史 Round 2 / Round 3 只保留演化过程，不是实施 Source of Truth。

---

## 当前执行链

```text
User Query
→ Query Parser
→ Data Contract Validation
→ Scope / Population Resolver
→ Comparison Window Pairing
→ Comparable Monthly Mask（若适用）
→ Canonical Analysis Input
→ Presence / Period State / Transition
→ Exhaustive Atomic Calculation
→ Atomic / Parent Decomposition
→ Closure / Reversal / Semantic Checks
→ Gross Movement / Offset
→ Individual Materiality
→ Collective Materiality
→ Structured Long-tail
→ Cross-View Non-Additivity Guard
→ Attention Ranking
→ Mathematical WHY
→ Hypothesis / Evidence
```

---

## Query Scope / Population

真实问题可以动态定义 Parent：

```text
超体
地采
某分部
可比门店
5-6月
Q2
618
YTD
```

原则：

> **先决定哪些事实属于这道题，再启动 Parent-dependent 计算。**

Scope 改变后，Parent Rate、Mix/Rate、Offset、Materiality、Collective Materiality 等必须重算。

### 可比口径

上游提供：

```text
门店 × 月份 × comparable_flag
```

正式规则：

> **本年逐月定资格，同一资格同时约束本年与去年同期；逐月取数后合并，再从头分析。**

禁止：

- 再用去年同期自己的 comparable_flag 二次筛选；
- 用期末可比名单回刷整个累计期；
- Current 筛可比而 Base 仍用全量；
- 先算全量 Contextual Result 再筛 Comparable。

### 时间窗口

YTD 不再视为特殊算法，而是 `Comparison Window` 的一个实例。

跨月必须：

```text
先聚合底层 Amount / Numerator / Denominator
→ 再重算 Rate / Mix-Rate / Offset / Materiality
```

不能把月度率或月度 Mix / Rate 直接相加。

---

## 当前核心数学

### GMV

```text
ΔGMV_i = G1_i - G0_i
```

### 毛利额：Continuing Standard

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

### Parent 毛利率

```text
Total Parent
↓ Non-standard Bridge
Standard Parent
↓ Exit / Entry
Continuing Standard Parent
↓ Mix / Rate
```

---

## Materiality / Decision Precision

GS-001：

> **Abnormality ≠ Materiality。**

GS-002：

> **Individual Insignificance ≠ Collective Insignificance。**

Materiality 必须在当前 Parent Context 下重算；Collective Materiality 必须对 Cohort 整体重算，禁止把 Atomic Materiality 相加。

---

## Cross-View Guard

同一现象可以同时被门店、渠道、店型、分部等 View 看见。

> **Projection Is Evidence, Not Additional Effect。**

当前硬规则：

- 跨 View Materiality 不相加；
- 不自动把多个 View 命中写成多个独立原因；
- 无法判断关系时标记 `OVERLAP_UNKNOWN`。

System Interaction Test 已证明“一个问题被多个 View 重复升级”是现实风险；Finding Consolidation 仍未正式冻结。

---

## 当前永久 Regression

已冻结：

- `GS001-F001` Extreme Rate × Low Materiality；
- `GS002-F001` Collective Materiality Blind Spot；
- `GS003-F001` Comparable Population Assembly；
- `GS003-F002` Arbitrary Comparison Window Recalculation；
- Net Zero ≠ Absent；
- Entry / Exit Time Reversal Mapping；
- Parent Bennett Zero-denominator Gate；
- Atomic GP Unified Closure。

原则：

> **Every Accepted Counterexample Becomes a Regression Test。**

---

## 当前不应无目的复杂化的部分

目前没有证据需要推翻：

- Symmetric Bennett；
- Continuing Mix / Rate；
- `Total → Standard → Continuing` Bridge；
- Atomic Attribution vs Parent Re-decomposition；
- Derived Ratio Never Rolls Up；
- Gross Movement / Offset；
- Mathematical WHY ≠ Causal WHY。

当前主要风险已经从“公式会不会错”逐步转向：

> **输入总体是否定义正确、多个正确规则组合后是否产生重复解释，以及 Attention 是否仍然简洁有效。**

---

## 数据安全

本仓库不保存真实经营数据、真实门店名称、内部账号、Token、敏感网络地址或未经脱敏的业务明细。

Golden / Regression 使用完全合成数据或匿名结构参数。

# Production System Contract V0.4：统一生产系统合同

> 状态：`CURRENT CANONICAL SYSTEM CONTRACT / BASELINE V1 CANDIDATE`  
> 生效日期：2026-08-29  
> 适用范围：单店损益 A 阶段（GMV → 毛利额 → 毛利率）。  
> 当前证据链：GS-001 Materiality、GS-002 Collective Materiality、Repository Integration Audit、System Interaction Side-effect Test、GS-003 Query Scope & Comparable Population，以及多轮 Baseline Freeze Review / Self-Falsification。  
> 目标：把 Query Scope / Population、数学 Kernel、数据状态、Roll-up、Materiality、Collective Materiality、Cross-View Guard 与解释边界收口到一份统一执行合同。

---

## 0. Source of Truth 与优先级

运行期实现按以下优先级读取：

1. **本文件 `production-system-contract.md`**：当前统一 Production Contract；
2. `docs/data-contracts/store-pnl-data-contract.md`：输入、Presence、Period State、字段口径；
3. `docs/methodology/query-scope-and-population-assembly.md`：Query Scope、Comparable Population、Comparison Window；
4. `docs/methodology/production-shadow-decomposition.md`：数学公式与 Shadow 细节；
5. `docs/methodology/materiality-gate.md`：GS-001 设计依据；
6. `docs/methodology/collective-materiality.md`：GS-002 设计依据；
7. `impact-ledger-and-rollup.md`、`roll-up-engine.md`、`skill-execution-architecture.md`：实现参考；
8. `docs/testing/test-sample-specification.md`：测试与 Freeze / Implementation Acceptance 治理；
9. Historical Review / Retrospective / Roadmap 仅用于理解演化，不是实施 Source of Truth。

若支持文档出现旧表述或遗漏，以本文件为当前运行规则入口；测试是否足以 Freeze / Accept Implementation，以 Test Sample Specification 的对应 Gate 为准。

> **一份运行合同可以引用多份证据文档，但运行规则只能有一个当前入口。**

---

## 1. 总体执行链

```text
User Query
→ Query Parser
→ Data Contract Validation
→ Scope / Population Resolver
→ Comparison Window Grain Validation
→ Comparison Window Pairing
→ Comparable Monthly Mask（若适用）
→ Canonical Analysis Input for this Query
→ Presence Preservation
→ Period State / Transition
→ Exhaustive Atomic Calculation
→ Atomic / Parent Decomposition
→ Mathematical Invariants
→ Gross Movement / Offset
→ Individual Materiality
→ Registered Cohort Resolution / Validation
→ Collective Materiality across Registered Views
→ Structured Long-tail
→ Cross-View Non-Additivity Guard
→ Attention Ranking
→ Mathematical WHY
→ Hypothesis / Evidence
```

核心纪律：

> **Scope / Population First, Contextual Calculation Second.**

> **Exhaustive Calculation, Selective Attention.**

---

## 2. Query Scope / Population Assembly

Parent 不是固定大盘。用户问题可以动态定义：

```text
time_window
organization
division
store
store_type
channel
comparable
其他已注册筛选条件
```

例如：

```text
这个月超体业绩为什么下滑？
这个月地采毛利率为什么下降？
5-6月整体毛利为什么下降？
1-7月可比门店表现怎么样？
```

筛选后的合法事实集合就是本次 Query Scope。

Scope 改变后必须重算：

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

禁止先在全量 Parent 计算这些 Contextual Result，再从结果表做过滤。

---

## 3. 可比 Population

可比状态来自上游业务表：

```text
门店 × 月份 × comparable_flag
```

系统不在本层重新推断“开业后14个月”的资格逻辑；状态表本身是业务事实输入。

`comparable_flag` 不是新的 Period State。

### 3.1 本年逐月 Pair 规则

对本年月份 `t`：

```text
E_t = 本年 t 月 comparable_flag = 1 的门店集合
Current_t = 本年 t 月 ∩ E_t
Base_t    = 去年同期 t 月 ∩ 同一个 E_t
```

遍历所有月份后：

```text
APPEND monthly Current/Base pairs
→ Comparable Canonical Analysis Input
→ 从头运行完整 Analysis Engine
```

硬规则：

> **本年逐月定资格，同一资格同时约束本年与去年同期；逐月取数后合并，再从头分析。**

禁止：

```text
使用去年同期自己的 comparable_flag 再筛一次
使用期末可比名单回刷整个累计期间
只筛 Current、不筛 Base
先跑全量 Contextual Result 再筛 Comparable
```

---

## 4. Comparison Window

时间统一建模为：

> **Comparison Window（比较时间窗口）**

V1 Canonical Input 的时间字段为：

```text
period = YYYYMM
```

因此新增硬规则：

> **Comparison Window resolution MUST NOT be finer than the Canonical Input time grain.**

即：当前 V1 原生支持单月以及由完整月份组成的月份集合，例如：

```text
单月
5-6月
Q2
YTD
最近N个月
任意合法月份集合
```

“618”“双11”等业务标签只有在以下任一条件成立时才可用于当前 V1：

1. 上游输入本身已经按当前 Canonical Grain 提供了该业务窗口；或
2. 已注册 Window 可以被**完整且唯一**地解析为一组完整 `YYYYMM` 单元。

Window Registration 不会凭空创造更细粒度事实。

禁止：

```text
把 6/1-6/18 等 Sub-month Window 从月度输入中平均分摊出来
按天数比例拆月
由 AI 推断月内销售分布
通过隐式日期假设构造不存在的日粒度事实
```

跨月时：

```text
先定义 Current Window 与 Base Window
→ 聚合底层 Amount / Numerator / Denominator
→ 重建该 Window 的分析事实
→ 完整重算 Contextual Result
```

因此：

```text
Window Rate Change != SUM(月度 Rate Change)
Window Mix         != SUM(月度 Mix)
Window Rate Effect != SUM(月度 Rate Effect)
Window Materiality != SUM(月度 Materiality)
Window Offset      != SUM / AVG(月度 Offset)
```

YTD 只是 Comparison Window 的一个常见实例。

---

## 5. 当前 Atomic Grain 与事实域

标准门店事实原子：

```text
时间 × 门店 × 渠道
```

门店类型、分部、大区等属于属性 / Hierarchy / Grouping，不额外制造事实轴。

当前原子渠道：

```text
地采
集采
万家
星选
```

派生渠道组：

```text
大集采 = 集采 + 万家
```

`大集采` 不是第五个 Atomic Channel；不得把 `集采 + 万家 + 大集采` 当互斥成员再次相加。

---

## 6. Presence 与 Period State

Full Outer Join 后必须先保存：

```text
base_key_present
current_key_present
```

再处理金额空值。

Period State 以 Data Contract 为机械定义：

```text
ABSENT
STANDARD
NET_ZERO_PRESENT
ZERO_GMV_NONZERO_GP
NEGATIVE_GMV
INVALID_OR_MISSING
OTHER_NONSTANDARD
```

硬规则：

```text
ABSENT ≠ NET_ZERO_PRESENT
```

只有：

```text
ABSENT → STANDARD = PURE_ENTRY
STANDARD → ABSENT = PURE_EXIT
```

Atomic Entry / Exit 只描述当前 Grain 的业务原子新增 / 退出，不自动等于新店开业 / 关店，也不等于“进入可比”。

---

## 7. GMV

```text
ΔGMV_i = G1_i - G0_i
ΣΔGMV_i = ΔGMV_parent
```

GMV 在 A 阶段主要回答 WHAT / WHERE，不需要额外复杂归因算法。

---

## 8. 毛利额：Symmetric Bennett

对 `STANDARD → STANDARD` 且两期 Rate 合法的原子：

```text
Scale_i = (G1_i - G0_i) × (r0_i + r1_i) / 2
Rate_i  = (r1_i - r0_i) × (G0_i + G1_i) / 2
Scale_i + Rate_i = ΔGP_i
```

统一原子事实：

```text
atomic_gp_effect_total_i = current_gp_i - base_gp_i
```

其他 Transition：

```text
PURE_ENTRY: Entry = current_gp
PURE_EXIT: Exit = -base_gp
NONSTANDARD_TRANSITION: Nonstandard = current_gp - base_gp
```

全 Parent：

```text
Σ atomic_gp_effect_total = Δ Parent GP
```

---

## 9. Atomic Attribution 与 Parent Re-decomposition

Atomic Attribution 回答：

> 哪些原子拉动 / 拖累 Parent？

其 Effect 可以向上 SUM，但仍是 Atomic Attribution Roll-up。

Parent Re-decomposition 回答：

> 当前 Parent 自己的毛利变化，是总体 GMV 还是总体毛利率造成？

必须聚合 Parent 后重新 Bennett。

> **Σ Atomic Scale/Rate ≠ Parent Bennett Scale/Rate 的语义。**

Parent Bennett 仅在：

```text
Parent G0 != 0
Parent G1 != 0
Parent Rate 两期合法
Parent Data Gate = PASS
```

时运行，否则 `BOUNDARY_STOP`。

---

## 10. Parent 毛利率：Total → Standard → Continuing

```text
R^T = Total Parent Rate
R^S = Standard-only Parent Rate
R^C = Continuing Standard Rate
```

```text
Nonstandard = (R1^T - R1^S) - (R0^T - R0^S)
Exit        = R0^C - R0^S
Entry       = R1^S - R1^C
```

Continuing Set 内重新归一化：

```text
w0_i = G0_i / Σ_C G0
w1_i = G1_i / Σ_C G1
Mix_i  = (w1_i - w0_i) × (r0_i + r1_i) / 2
Rate_i = (r1_i - r0_i) × (w0_i + w1_i) / 2
```

完整闭合：

```text
Δ Parent Rate = Nonstandard + Exit + Mix + Rate + Entry
```

若 Continuing Set 为空但两期 Standard Parent 均存在：

```text
Membership Replacement = R1^S - R0^S
```

禁止硬造普通 Mix / Rate / Entry / Exit。

---

## 11. Time Reversal

```text
Scale ↔ Scale
Rate ↔ Rate
Mix ↔ Mix
Nonstandard ↔ Nonstandard
Membership Replacement ↔ Membership Replacement
Entry ↔ Exit
```

正式断言：

```text
Effect_reverse(mapped_component) = -Effect_forward(component)
```

---

## 12. Roll-up 与 Contextual Metric

可加金额可 `SUM`。

Derived Ratio：

```text
先聚合 numerator / denominator
→ 当前 Context 重算
```

不得直接 Roll-up：

```text
毛利率
GMV同比
Contribution %
Offset Intensity
Interaction Intensity
Individual Materiality
Collective Materiality
```

硬规则：

> **Materiality Never Rolls Up；Materiality Must Be Recomputed in Context.**

> **Time Window Changes Context；Window Contextual Results Must Be Recomputed.**

---

## 13. Gross Movement 与 Offset

```text
Gross Movement = Σ|Effect_i|
Net Movement = |ΣEffect_i|
Offset Intensity = 1 - Net/Gross
```

Gross=0 时 Offset=N/A。

必须区分 Factor Offset 与 Unit Offset。

Offset 只说明内部正负运动抵消程度，不自动说明现实经营原因。

---

## 14. Individual Materiality

金额指标：

```text
absolute_effect_i = |Effect_i|
```

Rate 指标：

```text
R0 = P0/G0
R1 = P1/G1
ΔR = R1-R0

ΔR(-i)
= (P1-P1_i)/(G1-G1_i)
- (P0-P0_i)/(G0-G0_i)

Parent Rate Materiality Impact_i
= |ΔR - ΔR(-i)|
```

移除后分母无定义：

```text
materiality_status = N/A
materiality_reason = LEAVE_ONE_OUT_DENOMINATOR_UNDEFINED
```

Tiny Denominator 仍可为 STANDARD，并增加 warning；禁止改原值、封顶 Rate、加 epsilon、删行。

不变量：

> **Low-Materiality Dominance Prohibited.**

---

## 15. Collective Materiality 与 Registered Cohort

不变量：

> **Individual Insignificance ≠ Collective Insignificance.**

Collective Materiality 只在已注册、membership 确定、具有稳定业务语义的 Factor / Hierarchy / Derived View 上运行。

当前 V1 可直接注册的基础 View：

```text
分部
门店
店型
渠道
大集采（Derived View）
```

`门店规模带` 不再仅因为 Golden / Synthetic Scenario 中存在标签就自动视为 Production Registered View。只有在实际 Production 输入中存在权威 `store_size_band`，或存在版本化且唯一可执行的 membership predicate 后，才允许注册。

未来可扩展：大区、门店规模带、品类、品牌、供应商、费用科目、返利类型等，但必须先满足同一注册条件。

### 15.1 Registered 的正式含义

`Registered` 不是“文档里写了一个名字”。

一个 Cohort / View 进入 Production Registry 前，membership 必须能由以下至少一种来源唯一解析：

```text
Canonical Contract 中的确定性 predicate
版本化配置中的稳定 membership 定义
权威上游字段 / hierarchy / attribute
```

至少能够追踪：

```text
cohort_id / view_id
membership_rule 或 authoritative_source
source_version / rule_version
适用 grain / period
```

若 membership、来源或版本不能在计算前唯一确定：

```text
Registry Validation = FAIL
→ 不运行该 Cohort 的 Collective Materiality
```

### 15.2 Pattern Label 不创造成员

下列名称可以作为**已经确定 membership 的 Cohort**的描述标签：

```text
BROAD_SAME_DIRECTION
STRUCTURAL_MIGRATION
BROAD_RATE_DETERIORATION
BROAD_RATE_IMPROVEMENT
DISTRIBUTED_OFFSET
```

但 Pattern Label 本身不能自动创造 Cohort membership。

Baseline V1 不承诺自动发现任意 Pattern 并据此搜索成员；Automatic Cohort Discovery / Pattern Mining 属于未来能力。

禁止任意 `2^N` 原子组合搜索。

对 Rate Cohort `C`：

```text
Collective Impact_C = |ΔR - ΔR(-C)|
```

必须整组重算，禁止把 Individual Impact 相加。

---

## 16. Structured Long-tail

低重大性项目不能删除。

金额 Long-tail：聚合原始 signed Effect。

Rate Long-tail：将整个集合移除后重新计算 Parent Rate Impact。

至少保留：

```text
Long-tail Total
Long-tail by Registered Factor / Direction
```

只有 membership 已经满足 Registered Cohort Contract 的结构化 Cohort，才允许通过 Collective Materiality 升级为候选 WHY。

---

## 17. Cross-View Non-Additivity Guard

同一底层变化可同时被门店、渠道、店型、分部等 View 看见。

> **Projection Is Evidence, Not Additional Effect.**

禁止跨 View Materiality 相加，也禁止自动把多个 View 命中当成多个独立 Driver。

在 Finding Consolidation 正式冻结前：

```text
finding_relation = OVERLAP_UNKNOWN
```

用于无法判断是否同一现象的情况。

---

## 18. Attention

Attention 必须在完整计算和 Materiality 之后运行。

至少考虑：

```text
Mathematical Effect
Parent Materiality
Collective Materiality
Offset
Direction
Boundary / Warning
View / Context
```

当前不冻结固定重大性阈值；阈值只影响展示，不影响事实和数学结果。

---

## 19. Language Contract

允许：

```text
拉动 / 拖累 GMV / GP
Parent 毛利变化体现为规模 / 总体率影响
存量结构变化形成 Mix 影响
存在内部对冲
大量同类小变化合并后形成重大结构影响
全口径与可比口径出现差异
```

不允许仅凭 A 层直接声称：

```text
政策导致
竞争导致
返利能力下降
执行不到位
主动补偿
新店培育不佳
```

现实原因进入 Hypothesis → Evidence。

---

## 20. Production Invariants

正式实现至少检查：

```text
Population Assembly Before Context
Current/Base Population Pair Consistency
Comparison Window Grain Compatibility
Comparison Window Recalculation
Query Context Recalculation
Atomic GP Closure
Atomic Bennett Closure
Parent Bennett Closure
Parent Rate Bridge Closure
Continuing Mix / Rate Closure
Time Reversal with Mapping
Presence / Zero Separation
Derived Ratio No-Roll-up
Materiality Context Recalculation
Low-Materiality Dominance Prohibited
Registered Cohort Resolvability
Collective Blindness Prohibited
Cross-View Non-Additivity
Boundary Routing
Unrounded Closure
No Causal Overclaim
```

任何适用的硬不变量失败：不进入最终 AI Interpretation。

Decision-layer 不变量失败时，可以保留已验证的数学结果，但必须停止输出“主要 WHY”。

---

## 21. 最小结构化结果字段

至少保留：

```text
metric
unit_of_measure
grain
scope
query_scope
comparison_period
comparison_window
comparison_window_grain
population_rule
population_rule_version
view_type
grouping_level
parent_context

decomposition_type
component_set
algorithm_version
data_contract_version
metric_definition_version
input_fingerprint / source_data_version

base_state
current_state
transition_type
boundary_status
boundary_reason
closure_error

mathematical_effect
materiality_impact
materiality_rank
materiality_status

cohort_id
cohort_membership_rule / cohort_authoritative_source
cohort_rule_version / cohort_source_version
collective_pattern
collective_materiality_impact

warning_flags
attention_status
finding_relation
```

若使用可比口径，还应可追踪 comparable status source/version，但不得把可比资格写成全局永久原子状态。

---

## 22. Persistent 与 Contextual 边界

Persistent Atomic Ledger 只保存 Parent-independent 事实：Presence、GMV/GP/Δ、Atomic State/Transition、`atomic_gp_effect_total`、适用的 Atomic Bennett、Boundary Flags、Version/Fingerprint。

Contextual Engine 在 Query Scope 确定后重算：

```text
Parent Bennett
Continuing Mix/Rate
Rate Bridge
Contribution
Offset
Individual Materiality
Registered Cohort Validation
Collective Materiality
Structured Long-tail
Attention
```

> **Parent-dependent 判断不得被物化成全局永久标签。**

---

## 23. 当前尚未冻结的内容

```text
Materiality 固定阈值
Attention Top N / Pareto 阈值
Finding Consolidation / 跨 View 去重算法
Collective Cohort 最终优先级策略
Automatic Cohort Discovery / Pattern Mining
Interaction Shadow 是否保留
```

这些在通过 Golden / 历史回测 / Human Adjudication 前，不得写成 Production 事实。

---

## 24. Baseline Freeze 与 Implementation Acceptance 边界

Methodology Contract 可以在尚不存在完整 Production Engine 的情况下 Freeze，但前提是其数学、语义、状态路由、已知反例与最小组合行为已经有可复现证据。

Production Engine / Runner 的工程验收属于后续 `Executable Implementation Acceptance Gate`，不能被“以后再测”取消；也不能反过来要求一个尚不存在的 Engine 才允许冻结已经充分验证的方法合同。

正式 Gate 以：

`docs/testing/test-sample-specification.md`

为准。

一句话：

> **先冻结法律，再验收执法程序；两道门都要过，但验证对象不同。**

---

## 25. 当前总原则

> **先决定哪些事实属于这道题，再决定这些事实为什么变化。**

> **可比不是新的分析算法，而是由本年逐月状态驱动的 Population Filter。**

> **Comparison Window 不能比输入事实更细；Window 名称不会创造不存在的数据。**

> **Registered Cohort 必须先有唯一可追踪的 membership，再谈 Collective Materiality。**

> **算得对只是最低门槛；还必须知道什么对当前 Parent 真的重要。**

> **不同 View 可以同时解释同一事实，但不能因此制造多个不存在的独立问题。**

> **数学归因负责把数据解释到极限，现实因果仍然需要 Evidence。**
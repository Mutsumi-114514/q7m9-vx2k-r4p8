# Production System Contract V0.4：统一生产系统合同

> **Bundled Runtime Snapshot**：本文件是可安装 Skill 内置的运行合同快照，来源于 `docs/methodology/production-system-contract.md`，用于脱离原仓库安装时执行。若本包更新，应从 Canonical Contract 同步，而不是在此独立发明新规则。  
> 状态：`CURRENT CANONICAL SYSTEM CONTRACT / BASELINE V1 CANDIDATE`  
> 生效日期：2026-08-29  
> 适用范围：单店损益 A 阶段（GMV → 毛利额 → 毛利率）。

---

## 0. Installed Package Source of Truth

本安装包运行时优先级：

1. **本文件 `references/production-system-contract.md`**：统一 Production Contract；
2. `references/store-pnl-data-contract.md`：输入、Presence、Period State、字段口径；
3. `references/query-scope-and-population-assembly.md`：Query Scope、Comparable Population、Comparison Window；
4. `SKILL.md`：运行入口、输出与 Field Trial 纪律。

若支持文件与本文件冲突，以本文件为准。

> **一份运行合同可以有多份解释材料，但运行规则只能有一个当前入口。**

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

系统不在本层重新推断资格逻辑；状态表本身是业务事实输入。`comparable_flag` 不是新的 Period State。

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

禁止：使用去年同期自己的 comparable_flag 再筛一次、使用期末可比名单回刷累计期、只筛 Current 不筛 Base、先跑全量 Contextual Result 再筛 Comparable。

---

## 4. Comparison Window

V1 Canonical Input：

```text
period = YYYYMM
```

硬规则：

> **Comparison Window resolution MUST NOT be finer than the Canonical Input time grain.**

当前支持单月和由完整月份组成的月份集合，如 5-6月、Q2、YTD、最近 N 个月。

“618”“双11”等业务标签只有在上游输入本身已提供对应业务窗口，或已注册 Window 能完整唯一解析为当前 Grain 时才支持。

禁止从月度输入平均分摊、按天数拆分、AI 推断月内分布或隐式构造日粒度事实。

跨月时：

```text
先定义 Current Window 与 Base Window
→ 聚合底层 Amount / Numerator / Denominator
→ 重建该 Window 的分析事实
→ 完整重算 Contextual Result
```

因此 Window Rate、Mix、Rate Effect、Materiality、Offset 不能直接对月度结果 SUM / AVG。

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

`大集采 = 集采 + 万家` 是 Derived View，不是第五个 Atomic Channel。

---

## 6. Presence 与 Period State

Full Outer Join 后必须先保存：

```text
base_key_present
current_key_present
```

再处理金额空值。

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

GMV 主要回答 WHAT / WHERE。

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

Atomic Attribution 回答哪些原子拉动 / 拖累 Parent；其 Effect 可以向上 SUM，但仍是 Atomic Attribution Roll-up。

Parent Re-decomposition 回答当前 Parent 自己的毛利变化是总体 GMV 还是总体毛利率造成，必须聚合 Parent 后重新 Bennett。

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

不得直接 Roll-up：毛利率、GMV同比、Contribution %、Offset Intensity、Individual Materiality、Collective Materiality。

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

> **Low-Materiality Dominance Prohibited.**

---

## 15. Collective Materiality 与 Registered Cohort

> **Individual Insignificance ≠ Collective Insignificance.**

Collective Materiality 只在已注册、membership 确定、具有稳定业务语义的 Factor / Hierarchy / Derived View 上运行。

当前 V1 基础 View：

```text
分部
门店
店型
渠道
大集采（Derived View）
```

一个 Cohort / View 进入 Production Registry 前，membership 必须能由至少一种来源唯一解析：Canonical Contract 的确定性 predicate、版本化配置、权威上游字段 / hierarchy / attribute。

至少可追踪：

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

Pattern Label 可以描述已确定 membership 的 Cohort，但不能自动创造 membership；禁止任意 `2^N` 原子组合搜索。

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

至少保留 Long-tail Total，以及按 Registered Factor / Direction 的结构化 Long-tail。

---

## 17. Cross-View Non-Additivity Guard

> **Projection Is Evidence, Not Additional Effect.**

同一底层变化可同时被门店、渠道、店型、分部等 View 看见。

禁止跨 View Materiality 相加，也禁止自动把多个 View 命中当成多个独立 Driver。

无法判断关系时：

```text
finding_relation = OVERLAP_UNKNOWN
```

---

## 18. Attention

Attention 必须在完整计算和 Materiality 之后运行，至少考虑 Mathematical Effect、Parent Materiality、Collective Materiality、Offset、Direction、Boundary / Warning、View / Context。

当前不冻结固定重大性阈值；阈值只影响展示，不影响事实和数学结果。

---

## 19. Language Contract

允许：拉动 / 拖累 GMV / GP、Parent 毛利规模 / 总体率影响、存量 Mix、内部对冲、Registered Cohort 合计重大、全口径与可比口径差异。

不允许仅凭当前层直接声称：政策导致、竞争导致、返利能力下降、执行不到位、主动补偿、新店培育不佳。

现实原因进入：

```text
Mathematical WHY
→ Hypothesis
→ Evidence
→ Conclusion
```

---

## 20. Production Invariants

至少检查：

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

任何适用 Hard Invariant 失败：不进入最终 AI Interpretation。

Decision-layer 不变量失败时，可以保留已验证数学结果，但必须停止输出“主要 WHY”。

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

若使用可比口径，还应可追踪 comparable status source/version。

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

这些不得在没有证据时写成 Production 事实。

---

## 24. 当前总原则

> **先决定哪些事实属于这道题，再决定这些事实为什么变化。**

> **可比不是新的分析算法，而是由本年逐月状态驱动的 Population Filter。**

> **Comparison Window 不能比输入事实更细；Window 名称不会创造不存在的数据。**

> **Registered Cohort 必须先有唯一可追踪的 membership，再谈 Collective Materiality。**

> **算得对只是最低门槛；还必须知道什么对当前 Parent 真的重要。**

> **不同 View 可以同时解释同一事实，但不能因此制造多个不存在的独立问题。**

> **数学归因负责把数据解释到极限，现实因果仍然需要 Evidence。**

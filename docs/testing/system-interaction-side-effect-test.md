# System Interaction / Side-effect Golden Test V0.1

> 状态：`CURRENT TEST PROTOCOL`  
> 适用基线：完成 Repository Integration Audit 后的 Production System Contract。  
> 目的：不是继续验证单个公式，而是验证多个已经成立的规则组合后，会不会在 Attention / Finding / WHY 层产生新的副作用。

---

## 1. 为什么需要单独做系统副作用测试

GS-001 与 GS-002 分别引入：

```text
Individual Materiality
Collective Materiality
Structured Long-tail
```

再叠加既有：

```text
Offset
多 View Roll-up
Atomic Attribution
Parent Re-decomposition
Boundary / Warning
```

每条规则单独成立，不自动保证组合后仍然产生稳定、简洁、非重复的经营解释。

因此本测试回答：

> **局部正确规则组合后，系统级输出是否仍然正确？**

---

## 2. 测试场景必须同时包含

至少同时存在：

1. 一个 Tiny Denominator + Extreme Rate + Low Parent Materiality 原子；
2. 一群单体低 Materiality、但 Collective Materiality 高的同向小量迁移；
3. 一个真正重大、应该进入主要 WHY 的单体业务问题；
4. 两个方向相反的重大群体，制造 High Offset；
5. 同一个底层结构变化可被多个 View 同时观察；
6. 正 / 负 Structured Long-tail；
7. 至少一个 Derived View，例如 `大集采 = 集采 + 万家`；
8. 单月与 YTD 至少一处方向或优先级不同。

---

## 3. 必须保持的数学不变量

```text
Atomic GP Closure
Parent Bennett Closure
Parent Rate Bridge Closure
Continuing Mix / Rate Closure
Time Reversal Mapping
Derived Ratio No-Roll-up
Presence / Zero Separation
YTD Recalculation
```

这些任何一个失败，都属于数学 / 执行层回归，不进入 Attention 讨论。

---

## 4. Decision-layer 必测不变量

### 4.1 Low-Materiality Dominance Prohibited

极端 Rate 但 Parent Impact 很低的原子：

- 可以提示；
- 不得仅凭比例夸张排到主要 WHY 首位。

### 4.2 Collective Blindness Prohibited

大量单体影响很小，但同向 Cohort 影响重大：

- 不得全部埋进无结构“其他”；
- 必须在 Registered View 下重新计算 Group Materiality。

### 4.3 Materiality Context Recalculation

不得：

```text
SUM / AVG Atomic Materiality
→ Group Materiality
```

### 4.4 Cross-View Non-Additivity

不同 View 的结果：

- 不数值相加；
- 不自动声明独立 Driver；
- 关系不明时 `OVERLAP_UNKNOWN`。

### 4.5 Long-tail Integrity

低重大性项不能被删除；Structured Long-tail 重新变得重大时必须升级。

### 4.6 Offset Does Not Suppress Materiality

Parent Net 很小、高 Offset 时，重大正负因素仍必须保留。

---

## 5. 重点观测的副作用指标

本测试不仅记录 PASS / FAIL，还记录：

```text
candidate_finding_count
major_finding_count
view_count
cross_view_overlap_pairs
high_overlap_pair_count
unknown_overlap_count
long_tail_bucket_count
promoted_collective_count
demoted_individual_count
offset_intensity
```

以及：

```text
Finding Duplication Ratio
= 高度重叠且方向 / 语义近似的 Finding Pair 数
  / 全部 Finding Pair 数
```

当前不冻结“合理 Findings 必须 ≤ N”的硬阈值。

但如果同一个设计真值被多个 View 重复列为独立主要问题，则直接判：

```text
Decision Precision = FAIL
```

---

## 6. Finding Evidence Contract

每个候选 Finding 至少绑定：

```text
finding_id
metric
parent_context
view_type
grouping_level
factor
cohort_id
collective_pattern
mathematical_effect
materiality_impact
direction
attention_status
evidence_atom_ids / evidence_atom_set_hash
finding_relation
```

这样才能判断两个 Finding 是否只是同一底层原子集合的不同投影。

---

## 7. Cross-View Overlap 诊断

对两个 Finding `A / B` 的证据原子集合：

```text
S_A
S_B
```

诊断：

```text
Jaccard(A,B)
= |S_A ∩ S_B| / |S_A ∪ S_B|
```

该值只用于检测重叠，不直接自动决定两者是否同一个现实原因。

建议诊断标签：

```text
DISJOINT
PARTIAL_OVERLAP
HIGH_OVERLAP
IDENTICAL_ATOM_SET
```

若两个 Findings：

```text
HIGH_OVERLAP / IDENTICAL_ATOM_SET
+ 同方向
+ 同一 Mathematical WHY 层级
```

却被系统输出成两个“独立主要原因”，记录为副作用 Finding。

---

## 8. 反复升降级检查

Materiality / Collective Materiality 组合后，必须记录每个对象的决策路径：

```text
CALCULATED
→ individual materiality
→ individual attention
→ cohort membership
→ collective materiality
→ final attention
```

允许：

```text
单体低重大性
→ 群体重大
→ 以群体身份升级
```

不允许：

```text
同一对象被多个重叠 Cohort 反复升级
→ 最终产生多个重复主要 WHY
```

也不允许单体已经是主要 WHY 后，又在多个 Cohort 中重复贡献新的“独立”主要问题，却没有明确 Supporting / Overlap 关系。

---

## 9. 测试证据

失败时必须保存：

```text
Original Scenario
Exact Failing Findings
Overlap Matrix / Evidence Sets
Minimal Reproducer
Expected Decision Behavior
Actual Decision Behavior
```

若发现成立：

```text
Human Adjudication
→ ACCEPT
→ Production Fix
→ Permanent Regression
```

---

## 10. 当前不预设答案

本测试不提前假设最终 Finding Consolidation 必须采用：

- 聚类；
- 图算法；
- Jaccard 阈值；
- AI 语义合并；
- 固定 Top N。

先观察系统如何失败，再选最简单、可解释、可测试的修复。

原则：

> **先证明副作用真实存在，再为副作用增加复杂度。**

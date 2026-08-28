# Collective Materiality：集体重大性与小量迁移

> 状态：`CURRENT PRODUCTION ADDENDUM`  
> 生效日期：2026-08-29  
> 适用范围：单店损益 A 阶段及未来扩展到组织、渠道、商品、供应商、费用等已注册分析因素。  
> 来源：GS-002 Size & Structural Migration Golden Simulation。  
> 上游规则：`docs/methodology/materiality-gate.md`。

---

## 1. 核心问题

GS-002 证明：

> **Individual Insignificance ≠ Collective Insignificance**  
> **单体不重大 ≠ 集体不重大。**

一个 `门店 × 渠道` 原子对 Parent 的影响可能极小，但大量原子若同时发生同方向、同性质的变化，合并后可能形成重大经营影响。

典型形式：

```text
大量门店
每家只发生少量渠道份额迁移
单个 Atom Parent Materiality 很低
        ↓
这些迁移具有共同方向与共同业务语义
        ↓
合并后形成显著 Parent Effect
```

因此 Materiality 不能只在 Atomic Grain 计算一次后向上继承。

---

## 2. 新增正式原则：Materiality 必须按 Context 重新计算

Materiality 是 Contextual Metric（上下文指标）。

禁止：

```text
Atomic Materiality
→ SUM / AVERAGE
→ Group Materiality
```

也禁止：

```text
Atom 被判低重大性
→ 上层所有 View 自动继承“低重大性”
```

正确方式：

```text
在当前业务 View / Parent Context 下
重新聚合原始 Numerator / Denominator / Effect
→ 重新计算该 Group 对 Parent 的 Materiality
```

该原则与：

> **Derived Ratio Never Rolls Up**

完全一致。

新增对应规则：

> **Materiality Never Rolls Up；Materiality Must Be Recomputed in Context.**

---

## 3. 两条重大性路径

Production Attention 至少同时运行两类重大性检查。

### 3.1 Individual Materiality（单体重大性）

回答：

> 哪个单一原子 / 单一业务单元，独立足以显著改变 Parent？

对 Rate 指标继续使用：

```text
Parent Leave-One-Out Impact_i
= |ΔR - ΔR(-i)|
```

### 3.2 Collective Materiality（集体重大性）

回答：

> 是否存在大量单体影响很小、但具有共同方向或共同业务语义的变化，合并后显著影响 Parent？

对候选群体 `C`：

```text
R0(-C) = (P0 - ΣP0_i) / (G0 - ΣG0_i)
R1(-C) = (P1 - ΣP1_i) / (G1 - ΣG1_i)

Collective Parent Rate Impact_C
= |ΔR - (R1(-C) - R0(-C))|
```

金额指标则直接使用该 Cohort（群体）的聚合 Effect。

硬规则：

> **不得把单体 Leave-One-Out Impact 直接相加，必须对整个 Cohort 重新计算。**

---

## 4. Collective Micro-Movement（集体小量迁移）

正式定义：

> **大量单体均只发生较小变化，但变化在同一已注册分析因素上呈现显著同方向、同性质或同路径特征，并在聚合后形成重大 Parent Effect。**

它可以发生在任何已注册因素，而不只发生在门店。

当前及未来典型因素包括：

```text
组织：大区 / 分部 / 门店 / 店型 / 门店规模带
渠道：地采 / 集采 / 万家 / 星选 / 派生大集采
商品：品类1 / 品类2 / 品类3 / 品牌
供应商：供应商
费用：费用科目
返利：返利类型 / 供应商
```

可能的 Collective Pattern（集体模式）至少包括：

```text
BROAD_SAME_DIRECTION
广泛同向变化

STRUCTURAL_MIGRATION
结构迁移

BROAD_RATE_DETERIORATION
广泛率恶化

BROAD_RATE_IMPROVEMENT
广泛率改善

DISTRIBUTED_OFFSET
分散式对冲 / 补偿
```

Machine Code 只描述数学 / 结构现象，不自动生成现实因果。

---

## 5. 不允许任意组合爆炸

Collective Materiality 不等于穷举所有原子子集。

禁止：

```text
任意 2^N 原子组合搜索
```

因为这会造成：

- 组合爆炸；
- 数据挖掘式过拟合；
- 事后拼凑一个“看起来重大”的群体；
- 业务语义不可解释。

Production 只允许在：

> **已注册、具有稳定业务含义的 Factor / Hierarchy / Derived View**

上形成候选 Cohort。

例如：

- 渠道整体；
- `大集采 = 集采 + 万家`；
- 某店型；
- 某分部；
- 某门店规模带；
- 未来某品牌 / 品类 / 供应商。

不得为了寻找显著结果临时拼接没有业务语义的任意门店集合。

---

## 6. Collective Detection 的执行顺序

建议 Production 顺序：

```text
Exhaustive Calculation
→ Individual Materiality
→ Major Individual WHY
→ 对剩余低单体重大性项按 Registered Factors 重新组织
→ Direction / Migration Pattern Detection
→ Cohort Formation
→ Group Materiality Recalculation
→ Collective WHY / Long-tail
→ Attention Ranking
```

其中：

### 6.1 Direction Detection

至少检测：

- 同方向 GMV Change；
- 同方向 GP Change；
- 同方向 Rate Change；
- 权重向同一目标类别迁移。

### 6.2 Migration Detection

对于互斥或可解释的结构因素，如渠道：

```text
Source Share ↓
Target Share ↑
```

可形成候选 Migration Cohort。

例如：

```text
地采 → 大集采
```

只能表述为结构迁移，不自动表述为“政策导致”或“主动经营动作导致”。

---

## 7. Long-tail 必须保留结构，不应只有一个垃圾桶

`materiality-gate.md` 已要求低重大性项目不得静默删除，而应进入 Long-tail。

GS-002 进一步加固：

> **Long-tail 不应只有一个无结构总桶。**

原因：不同业务方向可能在总 Long-tail 中再次相互抵消。

因此至少保留：

```text
Long-tail Total
+ Long-tail by Registered Factor / Direction
```

例如：

```text
Long-tail：地采份额下降群
Long-tail：大集采份额上升群
Long-tail：小店GMV下降群
Long-tail：小店GMV增长群
```

若某个结构化 Long-tail Cohort 重新计算后达到重大水平，则必须升级为主要 WHY。

---

## 8. Collective Materiality 与 Offset 的关系

两者不能替代。

```text
Collective Materiality
回答：一群同类变化对 Parent 到底有多重要？

Offset
回答：多个正负变化之间有多少被相互抵消？
```

可能同时发生：

```text
渠道迁移群体重大
+ 大店Rate改善群体重大
+ 两个群体方向相反
→ Parent Net 很小
→ Offset 很高
```

因此 Parent 净变化小不能否定 Collective Materiality。

---

## 9. Decision Precision 新增不变量

新增：

### Collective Blindness Prohibited（禁止集体盲区）

若存在：

```text
多数单体 Materiality 很低
但某个具有明确业务语义的 Cohort 对 Parent 影响重大
```

而系统仅因为单体低重大性将其全部压入“其他”，则：

```text
Decision Precision = FAIL
```

### Context Recalculation Required（上下文重算必须）

任何 Group / View 的 Materiality 必须在该 Context 下重新计算，不能由 Atomic Materiality 汇总得到。

---

## 10. GS-002 实证记录

GS-002 中：

```text
Parent GMV Change ≈ -1.30%
Parent GP Change ≈ +2.92%
Parent Rate Change ≈ +0.7130pp
Mix ≈ -0.2837pp
Rate ≈ +0.9968pp
Store GMV Offset ≈ 89.5%
```

105 家较小门店共同发生渠道迁移：

```text
Group Parent Rate Materiality ≈ 0.4202pp
Median Atom Materiality ≈ 0.000267pp
Group / Median Atom ≈ 1575x
```

说明：

> **用 Atom Materiality 判断“这些变化不重要”是错误的；必须在渠道 / 门店群 / 派生渠道等业务 View 上重新计算。**

---

## 11. 当前 Production 执行链更新

```text
Data Contract Validation
→ Period State / Transition
→ Exhaustive Calculation
→ Atomic / Parent Decomposition
→ Closure / Reversal / Semantic Checks
→ Gross Movement / Offset
→ Individual Materiality
→ Collective Materiality across Registered Factors
→ Structured Long-tail Re-aggregation
→ Attention Ranking
→ Mathematical WHY
→ Hypothesis / Evidence
```

当前总原则升级为：

> **异常大不等于影响大；单体影响小也不等于集体影响小。**

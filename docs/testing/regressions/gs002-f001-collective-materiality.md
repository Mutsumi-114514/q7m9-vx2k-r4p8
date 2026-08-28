# Regression — GS002-F001 Collective Materiality Blind Spot

> Status: `PERMANENT_REGRESSION_CANDIDATE_ACCEPTED`  
> Source: `GS-002_SIZE_STRUCTURAL_MIGRATION`  
> Discovery Baseline: `12281978510e10507bd040900df3abdfd28aacfc`  
> Production Rule: `docs/methodology/collective-materiality.md`

---

## 1. Purpose

验证：

> 大量单体低重大性变化如果具有共同方向 / 共同结构迁移语义，并在聚合后形成重大 Parent Effect，系统必须识别该 Collective Effect，不能只看 Atomic Materiality 后把它们统一压入“其他”。

核心不变量：

```text
Individual Insignificance
≠
Collective Insignificance
```

以及：

```text
Materiality Never Rolls Up
Materiality Must Be Recomputed in Context
```

---

## 2. Canonical Reproducer

构造一个 Parent，包含大量较小业务单元。

Base：

```text
多数小单元：
高毛利渠道 A 权重较高
低毛利渠道 B 权重较低
```

Current：

```text
每个小单元仅迁移少量权重：
A ↓ 1~2pp
B ↑ 1~2pp
```

要求：

- 每个 Atom 的 Parent Leave-One-Out Impact 都很低；
- 迁移方向在大量单元中一致；
- 将该迁移 Cohort 作为一个 Group 重新计算后，对 Parent Rate 影响达到重大水平。

可以加入另一个方向相反的业务群体，使总体 Parent Net 仍然较小，以验证 Offset 不会掩盖 Collective Materiality。

---

## 3. Expected Behavior

系统必须：

1. 完整保留所有 Atomic Fact / Attribution；
2. 正常计算 Individual Materiality；
3. 不把单个低重大性标签向上 Roll-up；
4. 对 Registered Factor / View 重新组织低重大性项；
5. 识别同向 / 同性质 / 结构迁移 Cohort；
6. 对 Cohort 重新计算 Group Materiality；
7. 若 Group Materiality 重大，则升级为主要 WHY；
8. 若多个 Cohort 方向相反，则同时保留，并用 Offset 描述其相互抵消；
9. 不把结构迁移自动解释成现实因果。

允许的前台语言示例：

> 单个门店渠道变化均不显著，但大量门店同时从渠道 A 向渠道 B 迁移，合并后形成显著的结构性毛利率影响。

不允许：

> 这些门店单个都不重要，因此渠道结构没有重大变化。

---

## 4. Rate Materiality Expected Formula

单体：

```text
Impact_i = |ΔR - ΔR(-i)|
```

群体 `C`：

```text
Impact_C = |ΔR - ΔR(-C)|
```

其中：

```text
R0(-C) = (P0 - ΣP0_i) / (G0 - ΣG0_i)
R1(-C) = (P1 - ΣP1_i) / (G1 - ΣG1_i)
```

禁止：

```text
Impact_C = Σ Impact_i
```

因为 Leave-One-Out Materiality 属于 Contextual Metric，不能直接相加。

---

## 5. Registered Factor Constraint

不得通过任意组合搜索来创造一个“显著群体”。

Cohort 必须来自：

- 已注册业务 Factor；
- 已注册 Hierarchy；
- 已注册 Derived View；
- 明确机械规则定义的 Direction / Migration Pattern。

例如当前可使用：

```text
分部
门店
店型
门店规模带
地采 / 集采 / 万家 / 星选
大集采 = 集采 + 万家
```

未来扩展：

```text
品类1 / 品类2 / 品类3
品牌
供应商
费用科目
返利类型
```

---

## 6. Long-tail Failure Case

本 Regression 还必须测试：

> 一个单一无结构 Long-tail Bucket 可能把多个相反方向的 Cohort 再次抵消。

因此系统至少要保留：

```text
Long-tail Total
+ Structured Long-tail by Registered Factor / Direction
```

如果某 Structured Long-tail Cohort 重大，必须升级。

---

## 7. Failure Conditions

任一满足则 Regression FAIL：

```text
A. 将 Atomic Materiality SUM / AVG 成 Group Materiality
B. Atom 被判低重大性后，上层 View 自动继承低重大性
C. 大量同向小变化形成重大 Group Effect，但系统未发现
D. 所有低重大性项只进入一个无结构“其他”桶
E. 不同方向 Cohort 在总 Long-tail 中抵消后被错误判为无影响
F. 为发现群体效应穷举任意原子子集，产生无业务语义组合
G. Collective Effect 被直接解释成现实原因
```

---

## 8. GS-002 Discovery Record

完整 Golden Scenario 曾出现：

```text
Parent GMV Change ≈ -1.30%
Parent GP Change ≈ +2.92%
Parent Rate Change ≈ +0.7130pp
Mix ≈ -0.2837pp
Rate ≈ +0.9968pp
Store GMV Offset ≈ 89.5%
```

其中 105 家较小门店共同发生渠道迁移：

```text
Collective Parent Rate Materiality ≈ 0.4202pp
Median Atom Materiality ≈ 0.000267pp
Collective / Median Atom ≈ 1575x
```

该案例用于永久防止：

> **系统只会找“大单体”，却看不到“大量小动作共同形成的大变化”。**

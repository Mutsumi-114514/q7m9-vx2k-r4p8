# Roll-up Engine：带数学语义的多维聚合引擎

> 定位：Impact Ledger 之上的 L1 计算层。  
> 当前 V0.1 基座：`时间 × 门店 × 渠道`。  
> 核心目标：把原子事实按照不同业务视角安全地组织，同时保持指标数学语义、Parent Context、边界状态、单位和闭合关系不被破坏。

---

## 1. 一句话定义

Roll-up Engine 不是普通 `GROUP BY`，也不是简单透视表汇总。

> **Roll-up Engine = 带有指标数学语义、Parent Context、View Type 与 Unit 的多维聚合引擎。**

它负责回答：

- 什么可以直接 `SUM`；
- 什么必须重新计算；
- 什么只能在同一 Parent Context 下聚合；
- 什么属于 Parent-independent 原子事实；
- 什么属于 Contextual Metric；
- 什么时候只是 Atomic Attribution Roll-up；
- 什么时候必须 Parent Re-decomposition；
- 非标准业务怎样在不制造假率的前提下保持闭合；
- 不同单位的结果怎样被系统性禁止混用。

---

## 2. 四个基础动作

### 2.1 Filter：确定 Parent Scope

先明确当前问题的范围，例如：

```text
南京
A店
地采
202608
2026YTD
未来某大区
```

Parent Scope 决定相对指标的分母和解释上下文。

### 2.2 Group：选择观察层级

例如：

```text
门店
门店类型
渠道
月份
门店 × 渠道
分部
```

### 2.3 Aggregate：按照指标类型执行合法聚合

不能对所有指标统一 `SUM`。

引擎必须根据 Aggregation Metadata 选择：

- `SUM`；
- numerator / denominator 重算；
- 同一 Parent Context 下 contextual contribution SUM；
- Contextual decomposition 重算；
- Non-standard Bridge；
- Full Membership Replacement boundary。

### 2.4 Validate：闭合和语义检查

聚合后至少验证：

```text
Closure
View Type
Parent Context
Component Set
Unit of Measure
Boundary Status
Algorithm Version
Data Contract Version
Metric Definition Version
```

如果不能闭合、单位冲突或语义类型不合法，不继续输出 Mathematical WHY。

---

## 3. Aggregation Metadata：指标不是平等的

建议至少维护以下类型：

| 指标 / 结果 | 类型 | Roll-up 规则 |
|---|---|---|
| GMV | Additive Amount | `SUM` |
| 毛利额 | Additive Amount | `SUM` |
| GMV变化额 | Additive Amount | `SUM` |
| 毛利变化额 | Additive Amount | `SUM` |
| `atomic_gp_effect_total` | Parent-independent Additive Effect | `SUM`，闭合到 Parent ΔGP |
| 毛利率 | Derived Ratio | 聚合毛利 / GMV 后重算 |
| GMV同比 | Derived Ratio | 聚合本期 / 基期后重算 |
| Atomic Bennett Effect | Parent-independent Additive Effect | 同条件下可 `SUM`，但不得改名为 Parent Bennett |
| Parent Bennett Effect | Contextual Decomposition | Parent 改变后重算 |
| Continuing Mix / Rate | Contextual Additive Effect | 仅同一 Parent / Continuing Set / 比较期 / 算法下可 `SUM` |
| Entry / Exit Rate Effect | Contextual Bridge | Parent 改变后重算 |
| Non-standard Bridge | Contextual Bridge | Parent 改变后重算 |
| Gross Movement | Component-derived Scalar | 从当前 Component Set 的绝对值重新计算；单位继承目标 ΔY |
| Offset Intensity | Derived Ratio | 从当前 Component Set 重新计算，禁止相加 / 平均 |
| Interaction Intensity | Derived Ratio | 重新计算 |
| Contribution % | Contextual Ratio | 当前 Parent 下重新计算 |

核心规则：

> **Derived Ratio Never Rolls Up. Numerator / denominator / component set roll up; derived ratio is recalculated.**

---

## 4. Unit of Measure 是硬类型

建议至少维护：

| Machine Code | 中文标准名 | 典型用途 |
|---|---|---|
| `CNY_10K` | 万元 | GMV、毛利额、Bennett 金额影响 |
| `RATE` | 比率 | 内部计算中的率变化 |
| `PERCENTAGE_POINT` | 百分点 | 毛利率变化的人类展示 |
| `COUNT` | 数量 | 未来数量型指标 |

规则：

1. 不同 Unit 的结果禁止相加；
2. Gross Movement 与其目标 `ΔY` 使用同一 Unit；
3. 名称相似不代表 Unit 相同，例如 Atomic Bennett `Rate Effect` 是毛利金额影响，而 Continuing `Rate Effect` 是毛利率影响；
4. AI Interpreter 不得仅凭字段名推断单位。

---

## 5. 最简单的 Roll-up：金额加总

例如：

```text
A店 × 地采    GMV变化 -500
A店 × 集采    GMV变化 +100
---------------------------
A店           GMV变化 -400
```

继续：

```text
A店 -400
B店 -300
C店 +100
-------------
分部 -600
```

只要底层归属不重复，可加金额应始终闭合。

---

## 6. 不同投影不能再次相加

同一个 Parent 差异可以分别按：

- 门店；
- 渠道；
- 门店类型；

完整闭合。

但这些只是同一底层原子事实的平行投影。

例如：

```text
门店视角总计 = -1000
渠道视角总计 = -1000
```

不能再得到：

```text
-1000 + -1000 = -2000
```

> **投影是观察方式，不是额外原因。**

---

## 7. Rate / Derived Ratio 必须重算

假设：

```text
A店地采 毛利率 30%
A店集采 毛利率 10%
```

不能：

```text
A店毛利率 = 20%
```

除非两个渠道 GMV 恰好相同。

正确：

```text
A店毛利率 = A店全部渠道毛利额合计 / A店全部渠道GMV合计
```

同理：

- 禁止平均门店同比；
- 禁止平均月份毛利率；
- 禁止平均 Offset；
- 禁止相加 Contribution %；
- YTD 毛利率必须重新用累计 numerator / denominator 计算。

---

## 8. Atomic Attribution Roll-up 与 Parent Re-decomposition

这是两个必须严格区分的 View。

### 8.1 Atomic Attribution Roll-up

回答：

> 哪些原子业务单元拉动 / 拖累了当前 Parent？

完整的毛利额 Atomic Attribution 不只包含 Continuing Standard Bennett，还必须包含：

```text
Continuing Scale / Rate
Pure Entry
Pure Exit
Non-standard GP Effect
```

并满足：

```text
Σ Atomic Components
= Σ atomic_gp_effect_total
= Δ Parent GP
```

这仍然是原子影响向上汇总。

### 8.2 Parent Re-decomposition

回答：

> Parent 自己为什么变化，是总 GMV 还是总体毛利率？

必须：

```text
Parent = 当前对象
```

重新聚合 Parent 总 GMV / 毛利 / 率，再运行 Parent Bennett。

Parent Bennett 只有在两期 Parent GMV 非 0、Parent Rate 合法、数据状态通过校验时才允许执行。

所以：

> **Atomic Effect Roll-up ≠ Parent Total Scale / Rate。**

---

## 9. Presence / State 不得在 Roll-up 中丢失

底层原子必须区分：

```text
ABSENT
NET_ZERO_PRESENT
ZERO_GMV_NONZERO_GP
NEGATIVE_GMV
STANDARD
...
```

Roll-up 不能通过统一填 0 抹去 Presence。

尤其：

```text
ABSENT → STANDARD
```

才是自动 Pure Entry；

```text
NET_ZERO_PRESENT → STANDARD
```

不能因为基期净值为 0 就被改名成 Pure Entry。

Period State 的 Machine Code / 中文标准名由数据合同统一维护。

---

## 10. 毛利率分解的 Parent Context

当前毛利率 Production 采用：

```text
Total Parent
↓ Non-standard Bridge
Standard Parent
↓ Entry / Exit
Continuing Standard Parent
↓ Continuing Mix / Rate
```

### 10.1 Continuing Set 内部权重

只有两期均属于 STANDARD 的共同原子进入普通 Mix / Rate。

权重必须在 Continuing Set 内重新归一化：

```text
w0_i^C = G0_i / Σ(G0_j, j∈C)
w1_i^C = G1_i / Σ(G1_j, j∈C)
```

不能使用 Total Parent 权重。

### 10.2 换 Parent 必须整套重算

例如：

> “南京对华东一区毛利率变化的影响”

与：

> “南京自己毛利率为什么变化”

不是同一道题。

前者在华东一区 Parent 下计算 Contextual Contribution；后者将南京设为新 Parent 后重新做 Total / Standard / Continuing decomposition。

---

## 11. Non-standard Bridge

非标准业务不允许被悄悄赋予正常毛利率参与 Mix / Rate。

但它们不能简单丢掉，否则 Parent Closure 会断裂。

因此通过：

> **Non-standard Bridge Effect**

连接：

```text
Total Parent Rate
↕
Standard Parent Rate
```

这保证：

- 非标准真实金额不丢失；
- 不制造假率；
- Parent 毛利率仍可闭合。

如果未来要继续细分非标准业务，必须另做语义评审，不允许仅因字段名看起来像“调整”就自动归因。

---

## 12. Full Membership Replacement

若基期与本期 Standard Parent 都有业务，但 Continuing Standard Set 为空：

```text
C = ∅
```

则普通 Mix / Rate 不适用。

此时只计算：

```text
Standard Membership Replacement Effect
= R1^S - R0^S
```

再与 Non-standard Bridge 一起闭合 Parent 总体率变化。

机器必须明确输出“不存在共同存量原子”，不能硬造 Mix / Rate。

---

## 13. Time Reversal 的 Roll-up 约束

时间反转必须按 Component Mapping 验证：

```text
Scale ↔ Scale
Rate ↔ Rate
Mix ↔ Mix
Nonstandard ↔ Nonstandard
Replacement ↔ Replacement
Entry ↔ Exit
```

正式关系：

```text
Effect_reverse(mapped_component)
= -Effect_forward(component)
```

不能要求 Reverse Entry 与 Forward Entry 同名反号。

---

## 14. Gross Movement / Offset 的 Roll-up 规则

Offset 不是可以相加或平均的属性。

### Factor Offset

当前 Parent / View 下，先得到明确因素集合：

```text
Scale / Rate
或
Nonstandard / Exit / Mix / Rate / Entry
```

然后：

```text
Gross Movement = Σ|Effect_i|
Offset = 1 - |ΣEffect_i| / Gross Movement
```

Gross Movement 的 Unit 与这些 Effect 的目标 `ΔY` 相同。

### Unit Offset

当前 Parent 下，先确定业务单元集合，例如：

```text
门店A +100
门店B -100
```

再从这些单元影响重新计算 Unit Offset。

禁止：

```text
南京 Offset + 苏北 Offset
```

也禁止默认平均各店 Offset。

---

## 15. Roll-up 与 Drill-down 是同一层级系统的两个方向

从：

```text
分部
```

向下看门店、渠道，是 Drill-down；从原子向上，是 Roll-up。

它们不需要两套独立数学引擎，本质是同一个 hierarchy / grouping 模型在不同方向切换。

---

## 16. Attribute-based Roll-up

门店类型属于门店属性：

```text
门店 → 门店类型
```

可以得到：

```text
MALL
城市旗舰店
其他店型
```

但必须使用当期有效属性。

未来大区 / 分部归属变化同理。

---

## 17. 时间 Roll-up

金额：

```text
YTD Amount = Σ 月度 Amount
```

率：

```text
YTD Rate = ΣYTD Numerator / ΣYTD Denominator
```

Contextual decomposition：

> 先重建 YTD 的 Base / Current 原子存在性与状态，再重新跑 Parent decomposition。

禁止把月度 Mix / Rate 直接相加冒充 YTD WHY。

---

## 18. Attention 不属于 Roll-up Kernel

下列都属于 L2：

- Top N；
- 影响额排序；
- Pareto；
- 累计覆盖率；
- Gross Movement / Offset 组合筛选；
- 正负池；
- 金额阈值。

> **计算层穷举全部合法对象，Attention 层只负责挑选人真正需要看的对象。**

---

## 19. Roll-up Engine 的最小结构化输出

建议任何 Contextual Result 至少携带：

```text
metric
unit_of_measure
grain
scope
parent_context
comparison_period
view_type
grouping_level
component_set
algorithm_version
data_contract_version
metric_definition_version
input_fingerprint / source_data_version
boundary_status
boundary_reason
closure_error
```

这样机器不会仅凭一个数值或字段名跨 Context / Unit 使用。

---

## 20. 最终原则

> **Roll-up 不是把数字加起来，而是把带数学类型、单位和语义上下文的事实安全地重新组织。**

金额可以简单；率、归因、Offset、Entry / Exit、非标准业务都必须尊重自己的聚合语义。

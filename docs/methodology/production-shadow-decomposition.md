# Production / Shadow 分解算法基线

> 定位：A 阶段（GMV → 毛利额 → 毛利率）的当前有效算法基线。  
> 原子颗粒度：`时间 × 门店 × 渠道`。  
> 目标：保证闭合、公允、时间反转一致、作用域正确、边界显式、业务语义可解释，并把真正需要验证增量价值的内容留在 Shadow。

---

## 1. 当前默认指标口径

当前 V0.1 中，若用户只说“毛利 / 毛利率”而未额外限定：

> **默认指综合毛利 / 综合毛利率。**

当前默认分母：

```text
G = 不含仅双记 GMV
```

默认：

```text
综合毛利率 = 综合毛利 / 不含仅双记 GMV
```

如果用户明确指定商品毛利、前台毛利、后台毛利，则应切换到对应已验证恒等式和对应毛利率，不得在分析过程中静默换口径。

---

## 2. Grain 与 Factor 必须区分

`门店 × 渠道` 虽然由多个业务字段组成，但在当前计算内核中已经是一个原子业务坐标。

> **Grain Dimensionality（原子颗粒度由几个字段构成） ≠ Factor Dimensionality（归因模型中有几个待分配因素）。**

是否需要 Shapley，取决于目标指标数学模型中是否存在多个需要分配交互影响的因素，而不是原子主键由几个字段组成。

---

## 3. Algorithm Invariants（算法不变量）

Production 算法必须满足其适用的不变量：

1. **Closure（闭合）**：分解项严格闭合到真实变化；
2. **Order Independence（顺序无关）**：对存在多个数学因素且该性质有定义的算法，不能因先改变哪个因素而改变归因；
3. **Symmetry（对称性）**：对数学地位相同的因素，不被人为偏袒；
4. **Time Reversal Consistency（时间反转一致性）**：交换基期 / 本期后，按正式 Component Mapping（分量映射）反号；
5. **Scope Consistency（作用域一致性）**：换 Parent / View / 时间 Scope 后按规则重算；
6. **Semantic Validity（语义有效性）**：数学闭合但业务含义荒谬，仍判 FAIL；
7. **Atomic Reproducibility（原子可复算）**：同一输入与算法版本结果确定；
8. **No Causal Overclaim（不冒充现实因果）**：A 层只给 Mathematical WHY。

公式正确只是最低门槛；只有同时满足其适用的不变量，才允许进入 Production。

### 3.1 Invariant Applicability Matrix（不变量适用矩阵）

| 算法 / 组件 | Closure | Time Reversal | Symmetry | Order Independence |
|---|---:|---:|---:|---:|
| Symmetric Bennett | 必须 | 必须 | 必须 | 必须 |
| Continuing Mix / Rate | 必须 | 必须 | 必须 | 必须 |
| Entry / Exit Bridge | 必须 | 必须，按 Entry↔Exit 映射 | N/A | N/A |
| Non-standard Bridge | 必须 | 必须 | N/A | N/A |
| Membership Replacement | 必须 | 必须 | N/A | N/A |
| Gross Movement / Offset | 依赖其 Component Set 先闭合 | 继承 Component Set | N/A | N/A |

不允许对一个根本不存在“多个对称因素”的组件机械执行 Symmetry Test。

### 3.2 Time Reversal 的正式 Component Mapping

同名反号：

```text
Scale ↔ Scale
Rate ↔ Rate
Mix ↔ Mix
Nonstandard Bridge ↔ Nonstandard Bridge
Membership Replacement ↔ Membership Replacement
```

角色互换并反号：

```text
Entry ↔ Exit
```

正式断言：

```text
Effect_reverse(mapped_component)
= -Effect_forward(component)
```

因此禁止写成：

```text
reverse.entry == -forward.entry
```

正确应验证：

```text
reverse.exit  == -forward.entry
reverse.entry == -forward.exit
```

---

## 4. GMV：Production 不需要复杂算法

对每个原子单元 `i`：

```text
ΔGMV_i = GMV_1,i - GMV_0,i
```

同一 Scope 下：

```text
Σ ΔGMV_i = ΔGMV_parent
```

GMV 在 A 阶段主要回答 WHERE：

> 谁拉动 / 拖累了多少 GMV？

---

## 5. 毛利额 Production：Symmetric Bennett

基础恒等式：

```text
毛利额 = GMV × 毛利率
```

对“两期都属于标准业务、两期毛利率均具有合法语义”的原子，采用 **Symmetric Bennett Decomposition（Bennett 对称两因素分解：对称分配 GMV 与毛利率共同变化产生的交互影响，消除变化顺序偏见）**。

记：

- `G0 / G1`：基期 / 本期 GMV；
- `r0 / r1`：基期 / 本期毛利率。

### 5.1 GMV 规模影响

```text
Scale = (G1 - G0) × (r0 + r1) / 2
```

### 5.2 毛利率影响

```text
Rate = (r1 - r0) × (G0 + G1) / 2
```

### 5.3 闭合

```text
Scale + Rate = Δ毛利额
```

前台语言默认写成：

> GMV 规模变化拉动 / 拖累毛利 X；毛利率变化拉动 / 拖累毛利 Y。

Symmetric Bennett 同时满足 Time Reversal Consistency。

---

## 6. Period State 与 Transition：引用数据合同，不自行发明状态

Period State 的 Machine Code、中文标准术语与可执行定义以：

> `docs/data-contracts/store-pnl-data-contract.md`

为准。

核心状态包括：

| Machine Code | 中文标准名 |
|---|---|
| `ABSENT` | 无原子记录 |
| `STANDARD` | 标准业务状态 |
| `NET_ZERO_PRESENT` | 有记录但净额归零 |
| `ZERO_GMV_NONZERO_GP` | 零GMV非零毛利 |
| `NEGATIVE_GMV` | 负GMV状态 |
| `INVALID_OR_MISSING` | 数据缺失或无效 |
| `OTHER_NONSTANDARD` | 其他非标准状态 |

实现层至少分别保存：

```text
base_key_present
current_key_present
base_state
current_state
transition_type
```

### 6.1 Continuing Standard

```text
STANDARD → STANDARD
transition_type = CONTINUING_STANDARD
```

运行普通 Symmetric Bennett。

### 6.2 Pure Entry

只有：

```text
ABSENT → STANDARD
```

才能自动定义为：

```text
transition_type = PURE_ENTRY
Entry GP Effect = 本期毛利额
```

`NET_ZERO_PRESENT → STANDARD` 不属于 Pure Entry。

### 6.3 Pure Exit

只有：

```text
STANDARD → ABSENT
```

才能自动定义为：

```text
transition_type = PURE_EXIT
Exit GP Effect = -基期毛利额
```

`STANDARD → NET_ZERO_PRESENT` 不属于 Pure Exit。

### 6.4 Non-standard Transition

如果任一期间为 `NET_ZERO_PRESENT`、`ZERO_GMV_NONZERO_GP`、`NEGATIVE_GMV`、`INVALID_OR_MISSING` 或其他明确非标准状态，则普通 Bennett 不适用。

对于数据有效、金额可定义但算法语义非标准的原子：

```text
Non-standard GP Effect = 本期毛利额 - 基期毛利额
```

但不得把该金额强行解释成普通 Scale / Rate。

若状态为 `INVALID_OR_MISSING`，则不能为了金额闭合静默补值，必须进入 Boundary Stop / 数据修复路径。

---

## 7. Atomic GP Attribution：所有状态统一闭合

每个数据有效原子都必须先有一个稳定事实：

```text
atomic_gp_effect_total
= current_gp - base_gp
```

然后根据 Transition 分配到合法 Component Set：

```text
CONTINUING_STANDARD:
Scale + Rate = atomic_gp_effect_total

PURE_ENTRY:
Entry = atomic_gp_effect_total

PURE_EXIT:
Exit = atomic_gp_effect_total

NONSTANDARD_TRANSITION:
Nonstandard = atomic_gp_effect_total
```

因此同一 Scope 下必须满足：

```text
Σ atomic_gp_effect_total
= Δ Parent GP
```

以及：

```text
Σ(所有合法 Atomic Attribution Components)
= Δ Parent GP
```

这条总合同用于防止实现时只汇总 Bennett 原子而漏掉 Entry / Exit / Non-standard。

---

## 8. Atomic Attribution 与 Parent Re-decomposition 必须分开

> **原子 Attribution 结果可以向上 SUM，但 SUM 后不能自动继承“父级总体规模 / 总体率”的语义。**

### Atomic Attribution View（原子归因视图）

回答：

> 哪些 `门店 × 渠道` 拉动 / 拖累了 Parent？

标准存量原子的字段语义可写为：

```text
Atomic GMV Change Effect
Atomic Gross Margin Rate Change Effect
```

Entry / Exit / Non-standard 也必须作为同一 Atomic Attribution Component Set 的显式分量存在。

### Parent Re-decomposition View（父级重新分解）

回答：

> 这个 Parent 自己为什么变化，是总体 GMV 还是总体毛利率？

必须先聚合 Parent 的：

```text
总GMV
总毛利
总体毛利率
```

再运行 Parent Bennett。

> **数学可加总 ≠ 聚合后语义不变。**

### 8.1 Parent Bennett Preconditions（父级 Bennett 前置条件）

只有同时满足：

```text
Parent G0 != 0
Parent G1 != 0
Parent r0 / r1 均有合法定义
Parent 数据状态通过校验
```

才允许运行 Parent Bennett。

若任一不满足：

```text
Parent Bennett = N/A
boundary_status = BOUNDARY_STOP
boundary_reason = <明确原因>
```

禁止为了继续分解而把无定义的 Parent Rate 填成 0。

---

## 9. 毛利率 Parent Decomposition：三层状态桥接

毛利率分解不能把 Non-standard 业务既包含在 Entry / Exit 桥接里，又单独再算一次，否则会 double count（重复计算）。

因此正式拆成三层：

```text
Total Parent
↓ 去除 Non-standard
Standard Parent
↓ 去除 Entry / Exit，只保留共同存量
Continuing Standard Parent
```

记：

- `R0^T / R1^T`：基期 / 本期 **Total Parent Rate（全量总体率）**；
- `R0^S / R1^S`：基期 / 本期 **Standard Parent Rate（仅标准业务总体率）**；
- `R0^C / R1^C`：基期 / 本期 **Continuing Standard Rate（共同存量标准业务总体率）**。

### 9.1 Non-standard Bridge Effect（非标准业务桥接效应）

```text
Nonstandard Bridge
= (R1^T - R1^S) - (R0^T - R0^S)
```

含义：

> 被排除出普通率分解的非标准业务，整体使 Parent 总体率发生了多少变化。

在未验证更细语义前，不强行把该桥接拆成多个现实业务原因。

### 9.2 Exit Effect（退出效应）

如果共同存量集合非空：

```text
Exit Effect = R0^C - R0^S
```

含义：

> 从基期标准业务中移除后来退出的标准业务，对总体率造成多少影响。

### 9.3 Continuing Mix / Rate（存量结构 / 单元率效应）

```text
Continuing Effect = R1^C - R0^C
```

并拆成：

```text
Continuing Mix + Continuing Rate
```

关键硬约束：权重必须在 **Continuing Standard Set 内部重新归一化**：

```text
w0_i^C = G0_i / Σ(G0_j, j∈C)
w1_i^C = G1_i / Σ(G1_j, j∈C)
```

然后：

```text
Mix_i  = (w1_i^C - w0_i^C) × (r0_i + r1_i) / 2
Rate_i = (r1_i - r0_i) × (w0_i^C + w1_i^C) / 2
```

要求：

```text
ΣMix_i + ΣRate_i = R1^C - R0^C
```

不得使用 Total Parent 权重冒充 Continuing Set 权重。

### 9.4 Entry Effect（新增效应）

```text
Entry Effect = R1^S - R1^C
```

含义：

> 新进入标准业务加入后，相对于“仅共同存量标准业务”的状态，对总体率造成多少影响。

### 9.5 完整闭合

当相关率均有定义且共同存量集合非空：

```text
R1^T - R0^T
= Nonstandard Bridge
+ Exit
+ Mix
+ Rate
+ Entry
```

---

## 10. Full Membership Replacement 与空集合边界

如果基期和本期标准业务都存在，但：

```text
Continuing Standard Set = 空集
```

则 `R0^C / R1^C` 不存在，禁止硬算 Mix / Rate、Entry / Exit。

此时改为：

```text
Standard Membership Replacement Effect
= R1^S - R0^S
```

并报告：

> 基期与本期不存在共同存量标准原子，因此普通存量 Mix / Rate 不适用。

整体仍可闭合：

```text
Δ Parent Rate
= Nonstandard Bridge
+ Standard Membership Replacement Effect
```

如果 Total / Standard 分母本身为 0 或率无定义，则停止普通率分解并显式返回 Boundary / N.A.。

---

## 11. Gross Movement 与 Offset Intensity 进入 Production Kernel

对任意已经严格闭合、且 Component Set 明确的分解：

```text
ΔY = Σ Effect_i
```

定义：

```text
Gross Movement = Σ |Effect_i|
Net Movement   = |Σ Effect_i|
Offset Intensity = 1 - Net Movement / Gross Movement
```

含义：

- **Gross Movement（总运动量）**：内部因素到底发生了多大绝对运动；
- **Offset Intensity（对冲强度）**：这些运动中有多少被正负因素相互抵消。

当：

```text
Gross Movement = 0
```

则：

```text
Offset Intensity = N/A
```

而不是 `0%`。

### 11.1 Offset 必须绑定 Component Set 与 Unit

禁止只有一个裸字段：

```text
offset_intensity
```

至少区分：

- **Factor Offset**：例如 Scale / Rate，或 Nonstandard / Exit / Mix / Rate / Entry 等数学因素之间的对冲；
- **Unit Offset**：例如 Parent 下不同门店 / 渠道 / 原子单元影响之间的对冲。

结果至少绑定：

```text
scope
comparison_period
view_type
component_set
unit_of_measure
algorithm_version
```

Gross Movement 与其目标 `ΔY` 使用同一单位：

- 毛利额分解：万元；
- 毛利率分解：Rate / Percentage Point 语义；
- 不能把不同单位的 Gross Movement 混合比较或相加。

Gross Movement 必须执行：

```text
ABS before SUM
```

不能先 SUM 正负抵消后再取绝对值。

Offset 本身进入 Production；“多高应该进入 Attention”的阈值策略仍需历史回测。

---

## 12. Contribution % 不是基础稳定指标

Absolute Effect（绝对拉动 / 拖累额）是稳定核心。

Contribution % 依赖 Parent Net：

- Parent Net = 0：Undefined；
- Parent Net 很小且高对冲：贡献率可轻易出现数百 / 数千个百分点，业务解释不稳定。

因此 Contribution % 只属于 Parent Context 下的条件性展示指标，不写入 Atomic Ledger 作为永久核心事实。

---

## 13. Derived Ratio Never Rolls Up

可加金额可以 SUM。

派生比例 / 上下文指标不能直接 Roll-up，包括：

- 毛利率；
- Contribution %；
- Offset Intensity；
- Interaction Intensity。

必须先聚合其 numerator / denominator / component set，再在当前 Scope 下重新计算。

YTD 同理：

> 月度金额可 SUM；YTD Mix / Rate 必须先形成 YTD 原子状态，再重新分解。

---

## 14. Boundary Rules

### Negative Margin（负毛利率）

若 GMV 业务语义正常且毛利率可定义，Bennett 可以正常处理负毛利率。

### Negative GMV（负 GMV）

代数可计算不代表业务语义合法。当前归入 `NEGATIVE_GMV`，不静默进入普通 Mix / Rate；其 Parent 影响由 Non-standard Bridge 保证闭合。

### Net-zero Present（有记录但净额归零）

`NET_ZERO_PRESENT` 不等于 `ABSENT`。它不能自动制造 Pure Entry / Pure Exit，也不能制造 0% 毛利率。

### Tiny Denominator（极小分母）

保留原值并增加 small-base WARN；不在方法论阶段拍脑袋设金额阈值或缩尾。

### Invalid / Missing

必要字段缺失、类型错误或数据合同不满足时，不允许静默补值继续分解；进入 `BOUNDARY_STOP`。

### Display Rounding（展示舍入）

Closure Check 永远使用未舍入原值；禁止为了展示闭合而创造虚假 Residual / Other。

---

## 15. Shadow V1：显式 Interaction

Production Bennett 为低解释成本对称分配交互项。

Shadow 继续显式保留：

```text
Δ毛利额
= Base Scale
+ Base Rate
+ Interaction
```

```text
Base Scale  = ΔG × r0
Base Rate   = G0 × Δr
Interaction = ΔG × Δr
```

可计算：

```text
Interaction Intensity
= |Interaction|
  / (|Base Scale| + |Base Rate| + |Interaction|)
```

对于 Continuing Mix / Rate，也可在 Continuing Set 的归一化权重下显式保留 Interaction。

Shadow 的目标是验证显式交互是否：

- 改变问题优先级；
- 改变结论语言强度；
- 改变 Evidence 路径；
- 产生 Production 没有的增量发现。

如果连续真实月份没有增量价值，应删除或降级。

Offset 已不属于 Shadow，本身进入 Production Kernel。

---

## 16. Language Contract

A 阶段只允许 Mathematical WHY，不越权到现实 Causal WHY。

| 结果 | 可以说 | 不可以直接说 |
|---|---|---|
| ΔGMV | 拉动 / 拖累 GMV 多少 | 为什么销售下降 |
| Atomic Bennett | 原子规模 / 率数学影响 | 某业务动作导致 |
| Parent Bennett | 父级毛利变化体现为规模 / 总体率影响 | 现实经营根因 |
| Mix | 存量结构变化拉动 / 拖累多少 | 为什么结构变化 |
| Rate | 存量原子自身毛利率变化影响多少 | 为什么率变化 |
| Non-standard Bridge | 非标准业务整体影响多少 | 调整、退货等现实原因 |
| Offset | 存在内部对冲 | 存在经营风险 / 管理失误 |
| Entry / Exit | 新增 / 退出标准业务的数学影响 | 为什么新增 / 退出 |
| `ZERO_GMV_NONZERO_GP` | 零GMV下存在非零毛利金额 | 一定是返利 / 冲销 / 调整 |

现实原因继续进入 Hypothesis → Evidence。

---

## 17. 当前 Production / Shadow 状态

| 对象 | Production | Shadow |
|---|---|---|
| GMV | 原子 ΔGMV | 暂无必要 |
| 毛利额 | Continuing Standard 用 Symmetric Bennett；Pure Entry / Exit 单独列示；Non-standard Transition 只保留金额影响 | Explicit Interaction |
| 毛利率 | Total → Standard → Continuing 三层桥接；Non-standard + Exit + Continuing Mix/Rate + Entry | Continuing Mix / Rate 的显式 Interaction |
| Gross Movement | 基础派生指标，单位继承目标 ΔY | — |
| Offset Intensity | 基础派生指标，绑定 Component Set / Unit | Attention 阈值策略待历史回测 |
| Interaction Intensity | — | Shadow V1 |
| Contribution % | 条件性展示指标 | — |

当前总原则：

> **公式正确只是最低门槛；Production 还必须满足作用域、类型、聚合语义、边界处理与语言语义。**

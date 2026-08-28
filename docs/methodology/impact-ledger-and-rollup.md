# Impact Ledger 与向上聚合约束

> 定位：经营分析计算内核的稳定设计原则。  
> 当前 V0.1 应用范围：单店表 A 阶段，重点覆盖 GMV → 毛利额 → 毛利率；贡献利润仅保留浅层损益出口。  
> 当前最细稳定分析粒度：`时间 × 门店 × 渠道`。

---

## 1. 核心思想

经营分析不应先决定 `Top N`、累计覆盖 `X%` 或固定异常阈值，再反推底层数据结构。

第一优先级是建立可复核的最细粒度事实层，再在当前 Parent Context 下计算上下文归因，最后才做 Attention。

整体结构：

```text
L0  Persistent Atomic Ledger
    时间 × 门店 × 渠道
            ↓
L1  Contextual / Roll-up Engine
    当前 Parent 下的权重、分解、桥接、聚合
            ↓
L2  Attention
    排序 / Top N / Pareto / Offset / 自由筛选
```

核心原则：

> **Exhaustive Calculation, Selective Attention（计算穷举，注意力筛选）。**

---

## 2. Atomic Grain 与组织属性

当前稳定原子颗粒度：

```text
时间 × 门店 × 渠道
```

当前组织链：

```text
分部
└─ 门店
   └─ 渠道
```

未来可扩展：

```text
大区
└─ 分部
   └─ 门店
      └─ 渠道
```

门店类型属于门店属性 / 聚合路径，而不是新的原子事实轴：

```text
门店 → 门店类型
```

如果门店类型、分部归属、大区归属历史上发生变化，应使用当期有效映射，不能用今天的标签回刷历史。

---

## 3. Persistent Atomic Ledger 只保存 Parent-independent 事实

能脱离 Parent Context 独立成立的结果，可以物化进 Atomic Ledger。

每个比较对象至少应能得到：

```text
期间 / 比较期间
门店
门店类型（当期有效）
渠道

base_key_present
current_key_present

基期GMV
本期GMV
ΔGMV

基期毛利额
本期毛利额
Δ毛利额
atomic_gp_effect_total

基期毛利率
本期毛利率

base_state
current_state
transition_type
boundary_flags

unit_of_measure
grain
algorithm_version
data_contract_version
metric_definition_version
input_fingerprint / source_data_version
```

Period State Machine Code 与中文标准名以数据合同为准，当前至少包括：

| Machine Code | 中文标准名 |
|---|---|
| `ABSENT` | 无原子记录 |
| `STANDARD` | 标准业务状态 |
| `NET_ZERO_PRESENT` | 有记录但净额归零 |
| `ZERO_GMV_NONZERO_GP` | 零GMV非零毛利 |
| `NEGATIVE_GMV` | 负GMV状态 |
| `INVALID_OR_MISSING` | 数据缺失或无效 |
| `OTHER_NONSTANDARD` | 其他非标准状态 |

### 3.1 Presence 必须先于状态分类

Full Outer Join 后必须先保存：

```text
base_key_present
current_key_present
```

不能先把 Null 统一填成 0，再用 `GMV=0` 推断“没有业务”。

因此：

```text
ABSENT ≠ NET_ZERO_PRESENT
```

只有 Key 真正不存在时才能判 `ABSENT`。

### 3.2 原子毛利额统一闭合字段

所有数据有效原子都应保存：

```text
atomic_gp_effect_total
= current_gp - base_gp
```

并保证：

```text
Σ atomic_gp_effect_total
= Δ Parent GP
```

对于 `CONTINUING_STANDARD` 原子，还可保存：

```text
Atomic Bennett Scale Effect
Atomic Bennett Rate Effect
Atomic Gross Movement
Atomic Factor Offset Intensity
Atomic Closure Error
```

对于 Pure Entry / Exit / Non-standard，则保存对应合法金额组件。

---

## 4. Parent-dependent 结果不得永久写死进 Atomic Ledger

下列指标依赖当前 Parent / Scope / Comparison，不能作为全局永久原子属性：

- Parent GMV 权重；
- Parent Bennett；
- Continuing Mix / Rate；
- Entry / Exit Rate Effect；
- Non-standard Bridge；
- Parent / Unit Offset；
- Contribution %；
- Pareto / 累计覆盖率。

这些应在 Parent 确定后由 Contextual Engine 重新计算。

因此要明确区分：

```text
Persistent Atomic Ledger
≠
Contextual Contribution Ledger
```

后者可以临时 / 缓存保存，但必须绑定：

```text
parent_context
comparison_period
view_type
component_set
unit_of_measure
algorithm_version
data_contract_version
metric_definition_version
input_fingerprint / source_data_version
```

---

## 5. 原子影响金额与相对贡献必须分开

例如某 `门店 × 渠道` 的 GMV 变化为 `-500 万元`，这个金额属于该单元自身，可以被向上 SUM。

但：

> “它解释了 Parent 下降的 25%”

不是原子属性，因为 Parent 一变，分母就变。

因此：

> **原子台账保存绝对影响；贡献度、占父级拖累比例、累计覆盖率等相对指标按当前 Scope 重算。**

Contribution % 在 Parent Net 为 0 时无定义；Parent Net 很小且高对冲时可能出现数百 / 数千个百分点，因此只作为条件性展示指标。

---

## 6. 可加金额：安全 Roll-up

对于 Additive Amount（可加金额）：

```text
父级金额 = SUM(子级金额)
父级变化额 = SUM(子级变化额)
```

适用于：

- GMV；
- 毛利额；
- GMV 变化额；
- 毛利额变化额；
- `atomic_gp_effect_total`；
- 其他已确认可加金额。

不同投影视角可以分别完整闭合到同一 Parent，但不能再次相加。

---

## 7. Derived Ratio Never Rolls Up

核心硬规则：

> **Derived Ratio Never Rolls Up. Numerator / denominator / component set roll up; derived ratio is recalculated.**

包括：

- 毛利率；
- GMV 同比；
- Contribution %；
- Offset Intensity；
- Interaction Intensity。

例如总体毛利率：

```text
总体毛利率 = Σ毛利额 / ΣGMV
```

禁止平均门店、渠道、月份毛利率。

YTD 同理：

```text
YTD毛利率 = ΣYTD毛利额 / ΣYTD GMV
```

---

## 8. 毛利额 WHY：Atomic Attribution 与 Parent Re-decomposition

对 `CONTINUING_STANDARD` 原子：

```text
毛利额 = GMV × 毛利率
```

使用 Symmetric Bennett。

### Atomic Attribution View

回答：

> 哪些原子业务单元通过自身状态变化拉动或拖累 Parent？

完整 Atomic Attribution Component Set 不能只包含 Bennett 原子，还必须包含：

```text
Continuing Scale
Continuing Rate
Pure Entry GP Effect
Pure Exit GP Effect
Non-standard GP Effect
```

并满足：

```text
Σ(全部合法 Atomic Attribution Components)
= Σ atomic_gp_effect_total
= Δ Parent GP
```

同一 Parent Context 下可按门店、门店类型、渠道等 SUM，但 SUM 后只能叫 Atomic Effect Roll-up，不能自动改名为 Parent Bennett。

### Parent Re-decomposition View

回答：

> Parent 自己的毛利变化，是总 GMV 还是总体毛利率造成？

必须先聚合 Parent 总 GMV / 总毛利 / 总体毛利率，再重新跑 Parent Bennett。

Parent Bennett 必须通过前置条件：两期 Parent GMV 非 0、两期 Parent Rate 合法、数据状态通过校验。

> **Roll-up Attribution ≠ Parent Re-decomposition。**

---

## 9. 毛利额 Entry / Exit / Non-standard Transition

普通 Bennett 只适用于：

```text
STANDARD → STANDARD
```

### Pure Entry

只有：

```text
ABSENT → STANDARD
```

才自动定义：

```text
PURE_ENTRY
Entry GP Effect = 本期毛利额
```

### Pure Exit

只有：

```text
STANDARD → ABSENT
```

才自动定义：

```text
PURE_EXIT
Exit GP Effect = -基期毛利额
```

### Non-standard Transition

`NET_ZERO_PRESENT`、`ZERO_GMV_NONZERO_GP`、`NEGATIVE_GMV` 等不能被冒充为 ABSENT。

对数据有效的非标准状态迁移：

```text
Non-standard GP Effect = 本期毛利额 - 基期毛利额
```

若数据本身无效 / 缺失，则进入 Boundary Stop，不允许静默补值闭合。

---

## 10. 毛利率 WHY：只在当前 Parent 下计算

总体毛利率属于加权平均：

```text
R = Σ(w_i × r_i)
```

当前 Production 正式结构是：

```text
Total Parent
↓ Non-standard Bridge
Standard Parent
↓ Exit / Entry
Continuing Standard Parent
↓ Continuing Mix / Rate
```

### 10.1 Continuing Mix / Rate 的权重

只对 `Continuing Standard Set` 运行，并在该集合内部重新归一化：

```text
w0_i^C = G0_i / Σ(G0_j, j∈C)
w1_i^C = G1_i / Σ(G1_j, j∈C)
```

不能使用 Total Parent 权重冒充 Continuing Set 权重。

### 10.2 Parent Context 改变必须重算

例如：

- “南京对华东一区毛利率变化的影响”；
- “南京自己毛利率为什么变化”；

是两个不同问题。

前者是在华东一区 Parent 下 Roll-up；后者需要把南京设为新 Parent 重新分解。

---

## 11. Non-standard Bridge 保证异常业务不破坏 Parent Closure

非标准业务可能不能进入普通率分解，但只说“排除”是不够的，因为 Parent 总体率仍然包含这些真实金额 / 分母影响。

因此当前统一使用：

> **Non-standard Bridge Effect（非标准业务桥接效应）**

在 Total Parent 与 Standard Parent 之间保持闭合。

在没有额外可靠业务定义前，不把 Non-standard Bridge 再强行细分成多个现实原因。

---

## 12. Time Reversal 的 Component Mapping

时间反转不是要求所有同名字段机械反号。

同名反号：

```text
Scale ↔ Scale
Rate ↔ Rate
Mix ↔ Mix
Nonstandard Bridge ↔ Nonstandard Bridge
Membership Replacement ↔ Membership Replacement
```

角色互换：

```text
Entry ↔ Exit
```

正式断言：

```text
Effect_reverse(mapped_component)
= -Effect_forward(component)
```

---

## 13. Gross Movement 与 Offset

对任意已经闭合且 Component Set 明确的分解：

```text
Gross Movement = Σ |Effect_i|
Net Movement = |ΣEffect_i|
Offset Intensity = 1 - Net Movement / Gross Movement
```

当 `Gross Movement = 0`：

```text
Offset = N/A
```

Offset 必须说明：

```text
component_set
unit_of_measure
```

Gross Movement 与目标 `ΔY` 使用同一单位，不存在全局统一的“Amount”类型。

---

## 14. Attention 属于 L2，不污染计算内核

当 L0 / L1 已正确建立后，下列都只是查询 / 展示策略：

- Top N；
- 影响额排序；
- 累计覆盖 X%；
- Pareto；
- Offset 高对冲池；
- 正向 / 负向池；
- 金额阈值；
- 自由筛选。

原则：

> **先把所有合法对象计算到底，再决定当前问题需要看多少。**

---

## 15. 当前 A 阶段 V0.1 主干

```text
GMV
├─ ΔGMV 原子穷举
└─ WHERE Roll-up

毛利额
├─ CONTINUING_STANDARD：Atomic Bennett
├─ PURE_ENTRY / PURE_EXIT
├─ NONSTANDARD_TRANSITION
├─ Atomic GP Total Closure
├─ Atomic Attribution Roll-up
└─ Parent Re-decomposition + Preconditions

毛利率
├─ Non-standard Bridge
├─ Exit
├─ Continuing Mix
├─ Continuing Rate
├─ Entry
└─ Full Membership Replacement boundary

所有闭合分解
├─ Gross Movement
└─ Offset Intensity
```

当前阶段不进入客流、转化、件单价等 B 层 Driver，也不进入现实因果 WHY。

---

## 16. 最终不变量

1. `时间 × 门店 × 渠道` 是当前最细稳定原子坐标；
2. 计算层穷举，Attention 层筛选；
3. Presence 与 Zero 严格分离；
4. Parent-independent 事实才进入 Persistent Atomic Ledger；
5. Parent-dependent 指标必须绑定 Context 并重新计算；
6. 可加金额向上 SUM；Derived Ratio 不直接 Roll-up；
7. Atomic Attribution 与 Parent Re-decomposition 严格区分；
8. Mix / Rate 只对 Continuing Standard Set 运行；
9. Entry / Exit / Non-standard 必须走显式边界路径；
10. 不同投影视角不能重复相加；
11. Time Reversal 按 Component Mapping 测试；
12. 任何算法都必须同时通过其适用的 Closure、Reversal、Scope 与 Semantic Validity。

# Production / Shadow 分解算法第三轮评审：Execution Semantics

> **HISTORICAL REVIEW / NOT IMPLEMENTATION SOURCE**  
> 本文件用于完整保留第三轮评审过程与当时结论，其中 Counterfactual Bridge / Adjustment 方案已被后续 Repository Audit Repair 修正。  
> **实施时不得直接复制本文件公式；当前 Production 规则以 `production-shadow-decomposition.md` 和当前 Data Contract 为准。**

> 定位：A 阶段（GMV → 毛利额 → 毛利率）的第三轮算法与执行语义评审。  
> 前置基线：`production-shadow-decomposition.md`、`production-shadow-decomposition-review-round2.md`。  
> 原子颗粒度：`时间 × 门店 × 渠道`。  
> 本轮重点：假设公式正确、代码无 Bug、数据无误，继续攻击“机器是否仍可能因为执行语义错误而给出荒谬业务表达”。

---

## 1. 三轮评审的递进关系

第一轮解决：

> **怎么算才公平。**

第二轮解决：

> **算出来的数字在什么 Scope / Parent / View 下才有正确语义。**

第三轮解决：

> **如何防止机器即使严格执行公式，仍然做出人一眼就知道错误的操作或解释。**

因此第三轮不以寻找更复杂算法为目标，而以 Execution Semantics（执行语义）、Type（类型）、Invariant（不变量）和 Language Contract（语言合同）为核心。

---

## 2. Algorithm Invariants（算法不变量）升级为硬约束

当前基础算法至少必须满足：

1. **Closure（闭合）**：分解项严格闭合到真实变化；
2. **Order Independence（顺序无关）**：不能因先改变哪个因素而改变归因；
3. **Symmetry（对称性）**：数学地位相同的因素不被人为偏袒；
4. **Time Reversal Consistency（时间反转一致性）**：交换基期 / 本期后，各项影响只反号，不改变归因逻辑；
5. **Scope Consistency（作用域一致性）**：换 Parent / View / 时间 Scope 后必须按规则重算；
6. **Semantic Validity（语义有效性）**：即使数学闭合，只要业务语义荒谬，也判定 FAIL；
7. **Atomic Reproducibility（原子可复算）**：同一输入与算法版本结果确定；
8. **No Causal Overclaim（不冒充现实因果）**：A 层只允许 Mathematical WHY，不越权到现实 Causal WHY。

其中 Time Reversal Consistency 不再是“优雅性质”，而是基础算法的强制测试项。

> **Historical note：当前 Baseline 已进一步把 Time Reversal 细化为 Component Mapping；Entry / Exit 在反转时需要角色互换后反号，不能把本节的“同名反号”机械推广到所有组件。**

---

## 3. Offset Intensity 从 Shadow 升级为基础计算指标

第二轮将 Offset Intensity（对冲强度：净结果被内部正负运动相互抵消的程度）作为 Shadow 指标。

第三轮确认：Offset 本身是低成本、确定性、可普适到任意闭合分解的派生指标，因此应进入 Production Kernel（基础计算内核）。

对任意闭合分解：

```text
ΔY = Σ Effect_i
```

定义：

```text
Gross Movement = Σ |Effect_i|
Net Movement   = |Σ Effect_i|
Offset Intensity = 1 - Net Movement / Gross Movement
```

当 `Gross Movement = 0` 时：

> `Offset Intensity = N/A`，而不是 `0%`。

因为“没有任何运动”和“存在运动但没有对冲”不是同一语义。

### 为什么需要 Gross Movement

仅有 Offset 百分比会把“小额 100% 对冲”与“大额 100% 对冲”混在一起。

因此基础输出应同时保留：

- **Gross Movement（总运动量：内部因素绝对影响之和）**；
- **Offset Intensity（对冲强度：其中有多少被正负抵消）**。

Attention 层未来可研究 `Gross Movement × Offset Intensity` 的组合价值，但阈值策略仍需历史回测，不在基础公式中拍脑袋写死。

---

## 4. Offset 不能是一个“裸字段”：必须区分 Component Set

第三轮发现，同一个 Parent 可以同时存在不同类型的 Offset。

### Factor Offset（因素对冲）

例如 Parent Bennett：

```text
Scale +68
Rate  -72
Net    -4
```

此时：

```text
Factor Gross Movement = 140
Factor Offset Intensity ≈ 97.1%
```

回答：

> 规模与毛利率两个数学因素之间存在多强对冲？

### Unit Offset（单元对冲）

例如 Parent 下：

```text
A店 +100
B店 -100
Parent Net = 0
```

此时 Unit Offset = 100%，即使 Parent Bennett 的 Factor Offset 可能为 N/A。

回答：

> Parent 下业务单元之间有多少正负影响彼此抵消？

因此所有 Offset 至少必须绑定：

```text
scope
comparison_period
view_type
component_set
algorithm_version
```

禁止只保存：

```text
offset_intensity = 97%
```

而不说明对冲的究竟是因素、门店、渠道还是其他组成项。

### 聚合顺序硬约束

对冲诊断必须：

```text
ABS before SUM
```

即：先对组成项取绝对值，再求 Gross Movement。

禁止先把正负项 SUM 掉再取绝对值，否则会把真实内部运动消掉。

> **Historical note：当前 Baseline 已进一步要求 Offset / Gross Movement 绑定 `unit_of_measure`。**

---

## 5. Entry / Exit：第二轮方案 Semantic FAIL

第二轮曾尝试直接定义：

```text
Entry Rate Contribution_i = w1_i × r1_i
Exit Rate Contribution_i  = -w0_i × r0_i
```

虽然可以数学闭合，但第三轮构造低毛利新业务进入的反例后发现，该定义可能给出：

> “低毛利新业务拉动总体毛利率 +Xpp”

这样的业务荒谬表达。

因此判定：

> **Closure PASS，Semantic FAIL。第二轮 Entry / Exit 毛利率贡献公式废弃。**

---

## 6. Entry / Exit 改为 Counterfactual Bridge（反事实桥接）

设：

- `R0`：基期全部业务总体毛利率；
- `R0^S`：基期仅保留 Continuing Standard Atoms（存量标准原子）后的总体毛利率；
- `R1^S`：本期仅保留 Continuing Standard Atoms 后的总体毛利率；
- `R1`：本期全部业务总体毛利率。

定义：

```text
Exit Effect = R0^S - R0
Continuing Effect = R1^S - R0^S
Entry Effect = R1 - R1^S
```

其中 Continuing Effect 再使用 Symmetric Mix / Rate：

```text
Continuing Effect = Mix Effect + Rate Effect
```

于是：

```text
Δ总体毛利率
= Exit Effect
+ Continuing Mix Effect
+ Continuing Rate Effect
+ Entry Effect
```

业务语义：

- Exit Effect：退出业务离开后，相对于基期全量状态对总体率造成的影响；
- Entry Effect：新增业务进入后，相对于“仅存量业务”状态对总体率造成的影响；
- Mix / Rate：只解释两期都存在且率有定义的存量标准业务。

该桥接同时保持 Time Reversal Consistency：时间反转后 Entry / Exit 角色互换并反号。

### Full Membership Replacement（业务集合完全替换）

如果 Continuing Set 为空，则不存在 `R0^S / R1^S`，不得硬算 Mix / Rate。

应报告：

> 总体毛利率发生 Xpp 变化，但基期与本期不存在共同存量原子，因此存量 Mix / Rate 不适用。

> **DEPRECATED IMPLEMENTATION：后续 Repository Audit 发现本节与第 7 节 Adjustment Effect 联用时会发生 double count。当前已升级为 `Total → Standard → Continuing` 三层 Bridge，不得直接实现本节公式。**

---

## 7. Adjustment Effect 必须独立进入毛利率闭合

若某原子出现：

```text
GMV = 0
毛利 ≠ 0
```

则该原子不能制造虚假毛利率，也不能进入普通 Bennett / Mix / Rate。

但如果只排除它，又会破坏 Parent 毛利率分解闭合。

因此第三轮新增：

```text
Adjustment Contribution_t = Adjustment GP_t / Parent GMV_t
Adjustment Effect
= Adjustment Contribution_1 - Adjustment Contribution_0
```

最终 Parent 毛利率分解升级为：

```text
Δ Parent Margin Rate
├─ Exit Effect
├─ Continuing Mix Effect
├─ Continuing Rate Effect
├─ Entry Effect
└─ Adjustment Effect
```

要求严格闭合。

A 阶段不得自行猜测 Adjustment 的现实业务原因。

> **DEPRECATED IMPLEMENTATION：本节公式已被后续 Repository Audit 修正。当前 `ZERO_GMV_NONZERO_GP` 等非标准状态统一由 `Total → Standard` 的 Non-standard Bridge 处理。**

---

## 8. Mix / Rate 的合法作用域正式收紧

普通 Symmetric Mix / Rate 只允许运行于：

> **Continuing Standard Atoms（存量且标准的原子单元）**

至少满足：

```text
基期存在
本期存在
两期 GMV / 毛利率均具有合法业务语义
不是 Adjustment / Non-standard Atom
```

Entry、Exit、Adjustment 必须走自己的独立分支。

Negative GMV、极小分母等情况继续保留 Boundary Flag，不因为代数可计算就强行解释。

> **Historical note：当前 Data Contract 已将“存在”改为明确的 Key Presence，并把底层状态名改成机械 Machine Code，避免把净值 0 或业务猜测冒充状态事实。**

---

## 9. Contribution Ratio 降级为条件性展示指标

在高对冲场景：

```text
Scale +68
Rate  -72
Net    -4
```

若用 Net 作为分母，贡献率会变成：

```text
-1700%
+1800%
```

数学上合法，但业务解释价值极低。

因此：

> **Absolute Effect（绝对拉动 / 拖累额）是基础稳定输出；Contribution % 只属于 Parent Context 下的条件性展示指标。**

规则：

- Parent Net = 0：Contribution % = Undefined；
- Parent Net 很小：标记 Unstable Contribution；
- “多小算小”的阈值不在方法论阶段拍定，交给历史回测。

---

## 10. Derived Ratio Never Rolls Up

第二轮已有：

> Rate Never Rolls Up。

第三轮扩展为：

> **Derived Ratio Never Rolls Up（派生比例指标不得直接向上聚合）。**

包括但不限于：

- 毛利率；
- Contribution %；
- Offset Intensity；
- Interaction Intensity。

正确做法是：

> 先 Roll-up 它们的 numerator / denominator / component set，再在新 Scope 下重新计算。

因此不能：

```text
南京 Offset + 苏北 Offset
```

也不能默认平均门店 Offset。

---

## 11. Closure 必须使用未舍入值

展示层可以将金额或 pp 舍入，但：

> **Closure Check 永远使用未舍入原值。**

禁止因为展示层出现 `99.9` vs `100.0` 的舍入差异而人为补一个 `Other / Residual` 业务因素。

实现层优先使用稳定的 Decimal / 定点数策略属于工程问题，但方法论要求是：

> 展示舍入不得反向污染基础计算。

---

## 12. Language Contract（语言合同）

A 阶段允许 Mathematical WHY（数学 WHY），禁止冒充 Causal WHY（现实因果 WHY）。

| 结果 | 可以说 | 不可以直接说 |
|---|---|---|
| ΔGMV | 拉动 / 拖累 GMV 多少 | 为什么销售下降 |
| Atomic Bennett | 原子规模 / 率数学影响 | 某业务动作导致 |
| Parent Bennett | 父级毛利变化体现为规模 / 总体率影响 | 现实经营根因 |
| Mix | 结构变化拉动 / 拖累多少 | 为什么结构变化 |
| Rate | 存量单元自身毛利率变化影响多少 | 为什么率变化 |
| Offset | 存在明显内部对冲 | 存在经营风险 / 管理失误 |
| Entry / Exit | 新增 / 退出业务的数学影响 | 为什么新增 / 退出 |
| Adjustment | 调整项对率的数学影响 | 调整项的现实原因 |

无法由 A 层数据证明的内容继续进入 Hypothesis → Evidence，而不是由 AI 自行补故事。

> **Historical note：当前底层状态不再使用 `ADJUSTMENT` 作为 Machine Code，而使用 `ZERO_GMV_NONZERO_GP` 等机械状态名；“调整”只有在额外 Evidence 支持时才作为更高层业务标签。**

---

## 13. Skill 执行必须依赖 Type + Schema + Executable Invariant

不能只靠 Prompt 告诉 AI“请记住这些规则”。

结构化结果至少应携带：

```text
metric
scope
comparison_period
view_type
parent_context
decomposition_type
component_set
algorithm_version
boundary_status
closure_error
```

例如：

```text
view_type = atomic_attribution_rollup
```

系统就不允许把其因素字段解释为：

```text
parent_total_scale_effect
```

核心工程原则：

> **Code enforces（代码负责强制）**  
> **Schema constrains（结构负责约束）**  
> **Prompt explains（提示词负责解释）**

目标不是要求 AI“深刻理解”每个公式，而是让错误路径在系统里尽量不存在。

> **Historical note：当前执行架构已进一步加入 `unit_of_measure`、`grain`、数据 / 指标版本与 input fingerprint。**

---

## 14. 第三轮最终状态

| 项目 | 结果 |
|---|---|
| Symmetric Bennett | PASS，公式不改 |
| Atomic / Parent 双视图 | PASS，升级为硬类型约束 |
| Time Reversal Consistency | PASS，升级为硬不变量 |
| Gross Movement | PASS，进入基础计算内核 |
| Offset Intensity | PASS，从 Shadow 升级到基础计算内核 |
| 单一裸 Offset 字段 | FAIL，必须绑定 Component Set / View |
| 第二轮 Entry=`w1×r1` / Exit=`-w0×r0` | FAIL，废弃 |
| Entry / Exit | 改为 Counterfactual Bridge |
| Mix / Rate | 限定为 Continuing Standard Atoms |
| Adjustment 只排除不补桥 | FAIL，会破坏 Parent 毛利率闭合 |
| Adjustment Effect | 新增基础分解项 |
| Contribution % | 降级为条件性展示指标 |
| Derived Ratio Roll-up | 禁止，必须重算 |
| Interaction Intensity | 继续留在 Shadow |
| AI 自由解释 | 禁止，受 Language Contract 约束 |

第三轮最重要的结论：

> **数学正确只是最低门槛。机器执行系统必须同时满足作用域、类型、聚合语义、边界处理和语言语义，否则仍可能在“所有数字都对”的情况下输出错误经营结论。**

> **Historical snapshot only：表中 Counterfactual Bridge / Adjustment Effect 反映第三轮当时状态，当前实施必须以最新 Production Baseline / Data Contract 为准。**

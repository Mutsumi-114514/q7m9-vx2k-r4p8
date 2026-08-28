# Materiality Gate：重大性闸门

> 状态：`CURRENT PRODUCTION ADDENDUM`  
> 生效日期：2026-08-29  
> 适用范围：单店损益 A 阶段（GMV → 毛利额 → 毛利率）。  
> 来源：GS-001 Normal Baseline Golden Simulation 发现 `Extreme Rate × Low Materiality Attribution Problem`。  
> 优先级：在本规则与 `production-shadow-decomposition.md` 中 Tiny Denominator / Attention 相关旧表述冲突时，以本文件为准；后续统一合并回 Production Baseline。

---

## 1. 问题定义

GS-001 证明：

> **Closure（闭合）和 Time Reversal（时间反转）全部正确，并不保证 Attribution（归因）具有足够的 Decision Precision（决策解释精度）。**

典型情形：

```text
GMV 极小
+ GP 仍有少量数值
→ 原子毛利率极端
→ Continuing Mix / Rate 产生很大的正负分量
→ 分量彼此高度抵消
→ Parent 真实变化很小，但 WHY 看起来异常巨大
```

因此必须区分：

```text
Numerical Correctness   数值正确
Closure Correctness     闭合正确
Semantic Validity       语义合法
Decision Precision      是否值得据此做经营判断
```

前三项通过，不自动等于第四项通过。

---

## 2. 核心决策：不修改数学分解，新增 Materiality Gate

当前 Production 算法保持：

- Symmetric Bennett 不变；
- Parent `Total → Standard → Continuing` Bridge 不变；
- Continuing Mix / Rate 公式不变；
- Entry / Exit / Non-standard Bridge 不变；
- Closure 与 Time Reversal 合同不变。

新增一层：

```text
Validity
数据是否合法
    ↓
Decomposition
数学上完整分解
    ↓
Materiality Gate
这些结果对 Parent 到底有多大实际影响
    ↓
Attention
哪些结果值得进入主要 WHY / 提醒
```

原则：

> **计算穷举，不因低重大性删除事实；展示筛选，不允许低重大性极端比率劫持主要 WHY。**

---

## 3. `STANDARD` 与 `Tiny Denominator` 的关系

`Tiny Denominator（极小分母）` 不再被设计成新的 Period State，也不因为分母小而修改原始数值。

只要满足 Data Contract：

```text
key_present = true
GMV > 0
必要字段有效
rate 可数学定义
```

仍可保持：

```text
period_state = STANDARD
```

同时允许增加诊断字段：

```text
tiny_denominator_warning = true | false
extreme_rate_warning = true | false
```

硬规则：

> **Tiny Denominator 是风险信号，不是重大性的定义。**

禁止：

- 以固定绝对金额直接把原子改成 Non-standard；
- 把毛利率封顶 / 缩尾；
- 给分母机械加 epsilon；
- 删除极小 GMV 原子；
- 仅因 `|Rate| > 100%` 就判定该原子不重要。

一个 `-120%` 毛利率原子如果规模重大，仍可能是必须优先关注的真实经营问题。

---

## 4. Materiality 的定义：看对 Parent 的实际影响

重大性回答的不是：

> “这个原子自身看起来有多异常？”

而是：

> **“这个原子的存在 / 变化，实际改变了 Parent 结果多少？”**

因此正式区分：

```text
Abnormality ≠ Materiality
异常程度 ≠ 重大性
```

### 4.1 金额指标

对于 GMV、毛利额等可加金额，基础重大性事实优先使用：

```text
absolute_effect_i = |Effect_i|
```

并绑定：

```text
metric
scope
parent_context
comparison_period
unit_of_measure
component_set
```

不得用极端同比 / 极端比例替代绝对影响。

### 4.2 Rate 指标：Parent Leave-One-Out Impact

对毛利率等 Derived Ratio，推荐使用 **Parent Leave-One-Out Impact（移除该原子后 Parent 变化量改变多少）** 作为重大性事实。

Parent 两期：

```text
R0 = P0 / G0
R1 = P1 / G1
ΔR = R1 - R0
```

移除原子 `i` 后：

```text
R0(-i) = (P0 - P0_i) / (G0 - G0_i)
R1(-i) = (P1 - P1_i) / (G1 - G1_i)
ΔR(-i) = R1(-i) - R0(-i)
```

定义：

```text
Parent Rate Materiality Impact_i
= |ΔR - ΔR(-i)|
```

含义：

> 如果没有这个原子，Parent 毛利率的两期变化会改变多少个百分点。

若移除后任一期 Parent 分母为 0 或率无定义：

```text
materiality_status = N/A
materiality_reason = LEAVE_ONE_OUT_DENOMINATOR_UNDEFINED
```

禁止硬填 0。

注意：Leave-One-Out Impact 是 Contextual Metric（上下文指标），不可跨 Parent 直接 Roll-up。

---

## 5. Materiality 不负责重新分配数学归因

Materiality Gate 的职责是：

- 排序；
- 过滤主要 WHY；
- 调整展示层级；
- 识别“比例极端但 Parent 影响很小”的项目。

它**不负责**：

- 修改 Scale / Rate / Mix 的数学值；
- 为了看起来合理而重新分摊 Closure；
- 把低重大性项目从正式计算结果中删除。

因此必须同时保留：

```text
mathematical_effect
materiality_impact
attention_status
```

而不是只保留最终展示结果。

---

## 6. Extreme Rate × Low Materiality 的正式处理

当一个原子满足：

```text
Rate 异常 / 极端
AND
Parent Materiality 很低
```

允许输出：

```text
attention_status = LOW_MATERIALITY_WARNING
```

前台语言应接近：

> 该原子毛利率异常，但对当前 Parent 总体结果影响较小，保留提示，不列为主要变化来源。

不得输出：

> Parent 毛利率主要由该极端毛利率原子驱动。

除非 Materiality 结果支持这一结论。

负向异常可以在**同等 Materiality** 下提高 Attention 优先级用于防微杜渐，但负号本身不能覆盖重大性原则。

---

## 7. Long-tail Bucket：低重大性不能被静默丢弃

禁止：

```text
单个不重大
→ 全部删除
```

因为多个小项可能形成重大合计影响。

低重大性原子不单独展示时，必须保留 Long-tail（长尾）集合。

### 7.1 金额指标

长尾金额按原始 Effect 重新聚合：

```text
LongTail Effect = Σ Effect_i, i∈L
```

并保留正 / 负方向。

### 7.2 Rate 指标

不能把单个 Leave-One-Out Impact 直接相加。

应把整个长尾集合 `L` 作为一个 Group 重新计算：

```text
R0(-L) = (P0 - ΣP0_i) / (G0 - ΣG0_i)
R1(-L) = (P1 - ΣP1_i) / (G1 - ΣG1_i)

LongTail Parent Rate Impact
= |ΔR - (R1(-L) - R0(-L))|
```

若长尾集合整体变得重大，则必须升级为一个独立的 WHY：

> **广泛的小幅同向变化 / 长尾变化形成了重大总体影响。**

---

## 8. Materiality Threshold 当前不拍脑袋冻结

V0.1 当前冻结的是**重大性计算方法与决策顺序**，不是某个固定阈值。

暂不写死：

```text
GMV < X 元
Rate Impact < X bp
Effect < Parent 的 X%
```

阈值必须通过：

- Golden Scenario；
- 历史月份回测；
- 人工分析师判断一致性；
- False Positive / False Negative 对比；

后再进入 Production Config。

因此当前结果应同时保留连续值：

```text
materiality_impact
materiality_rank
```

阈值只影响 Attention，不影响事实与数学结果。

---

## 9. Materiality 与 Offset 必须同时观察

Materiality 与 Offset 回答不同问题：

```text
Materiality：某个原子 / 集合实际改变 Parent 多少？
Offset：内部正负运动有多少被相互抵消？
```

一个 Parent 可以同时存在：

```text
Parent Net 很小
+ Gross Movement 很大
+ 多个重大正负项彼此抵消
```

因此不能因为 Parent 净变化小，就自动认为所有子项不重大。

同样，不能因为某个数学 Component 极大，就自动认为其业务意义重大；必须回看 Parent Materiality。

---

## 10. Decision Precision Invariant

新增 Production 解释不变量：

> **Low-Materiality Dominance Prohibited（禁止低重大性支配主要解释）。**

正式要求：

1. 数学分解可以完整保留所有原子；
2. 主要 WHY 排序必须结合 Parent Materiality；
3. 极端 Rate 不能仅凭数值大小进入主要 WHY；
4. 被隐藏的低重大性项目必须进入 Long-tail Bucket；
5. Long-tail 一旦整体重大必须重新升级；
6. Attention 输出必须可追溯回原始 Mathematical Effect 与 Materiality Impact。

若出现：

> Parent 实际变化很小，主要 WHY 却被一个对 Parent 几乎无影响的 Tiny Denominator 原子占据

则即使 Closure / Reversal 全部 PASS，仍判：

```text
Decision Precision = FAIL
```

---

## 11. GS-001 对 Production 的裁决

GS-001 发现：

```text
Parent ΔRate ≈ -0.1691pp
Continuing Mix ≈ -1.5816pp
Continuing Rate ≈ +1.4017pp
Mix / Rate Offset ≈ 93.97%
```

Closure 与 Time Reversal 通过，但 Tiny Denominator 原子导致解释层出现明显 Attribution Instability。

当前裁决：

```text
Finding = ACCEPT
Root Cause = Extreme Rate × Low Materiality attribution pollution
Production Change = ADD MATERIALITY GATE
Math Formula Change = NO
Period State Change = NO
Raw Data Modification = NO
```

该 Finding 后续必须进入永久 Regression。

---

## 12. 当前执行顺序

最终 A 阶段执行链：

```text
Data Contract Validation
→ Period State / Transition
→ Exhaustive Calculation
→ Atomic / Parent Decomposition
→ Closure / Reversal / Semantic Checks
→ Gross Movement / Offset
→ Parent Materiality Evaluation
→ Long-tail Re-aggregation
→ Attention Ranking
→ Mathematical WHY
→ Hypothesis / Evidence
```

总原则升级为：

> **算得对只是最低门槛；还必须知道什么对总体真的重要。**

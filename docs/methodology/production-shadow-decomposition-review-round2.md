# Production / Shadow 分解算法第二轮评审

> **HISTORICAL REVIEW / NOT IMPLEMENTATION SOURCE**  
> 本文件用于完整保留第二轮评审过程、反例与当时结论。部分公式已在后续评审中被废弃。  
> **实施时不得直接复制本文件公式；当前 Production 规则以 `production-shadow-decomposition.md` 和当前 Data Contract 为准。**

> 定位：A 阶段（GMV → 毛利额 → 毛利率）第二轮算法压力测试。  
> 前置基线：`production-shadow-decomposition.md`。  
> 原子颗粒度：`时间 × 门店 × 渠道`。  
> 本轮重点：不再只测试单个原子的公式，而是测试 **Roll-up（向上聚合）后的语义、公允性、边界处理与 Shadow 增量价值**。

---

## 1. 第二轮最重要的新发现

第一轮已经证明：对单个原子单元，Symmetric Bennett（Bennett 对称两因素分解）可以把：

```text
毛利额 = GMV × 毛利率
```

严格拆成：

- GMV 变化影响；
- 毛利率变化影响；

并满足原子级闭合：

```text
Scale_i + Rate_i = Δ毛利额_i
```

第二轮进一步发现：

> **原子 Bennett 结果向上 SUM 后，虽然仍然数学闭合，但其“Scale / Rate”语义不一定等于把父级对象重新聚合后再做 Bennett 的“总体规模 / 总体毛利率”语义。**

这不是算法错误，而是两个不同问题：

1. **Roll-up Attribution View（原子归因向上聚合视图）**：这些原子业务单元的 GMV 变化与自身毛利率变化，合计如何影响父级毛利；
2. **Parent Re-decomposition View（父级重新分解视图）**：父级总体毛利变化，究竟来自父级总 GMV 变化，还是父级总体毛利率变化。

因此第二轮最核心的约束是：

> **Roll-up Attribution ≠ Parent Re-decomposition。两者都正确，但回答的问题不同，Skill 不得混用标签。**

---

## 2. 纯结构迁移压力测试：为什么必须区分两个 View

构造两个原子单元，毛利率完全不变，只发生 GMV 在高毛利与低毛利单元之间的迁移：

| 原子 | 基期GMV | 本期GMV | 基期毛利率 | 本期毛利率 |
|---|---:|---:|---:|---:|
| A（高毛利） | 100 | 50 | 30% | 30% |
| B（低毛利） | 100 | 150 | 10% | 10% |

父级：

```text
总GMV：200 → 200（完全不变）
总毛利：40 → 30
总体毛利率：20% → 15%
```

### 2.1 Atomic Bennett 向上 Roll-up

A：

```text
GMV变化影响 -15
毛利率影响   0
毛利变化    -15
```

B：

```text
GMV变化影响 +5
毛利率影响   0
毛利变化    +5
```

合计：

```text
原子GMV变动影响 -10
原子率变动影响    0
毛利变化         -10
```

数学完全正确，每一行也分别闭合。

但如果把 `-10` 直接说成：

> “父级总 GMV 规模下降拖累毛利 10”

就是错误语言，因为父级总 GMV 根本没有下降。

真正发生的是：

> GMV 从高毛利单元迁移到低毛利单元。

因此 Atomic Roll-up 下应使用更准确的术语：

> **Atomic GMV Change Effect（原子 GMV 变动影响：各原子单元 GMV 增减在各自盈利水平下形成的毛利影响，可能包含结构迁移）。**

### 2.2 Parent Re-decomposition

把父级重新视为：

```text
毛利 = 父级总GMV × 父级总体毛利率
```

则：

```text
父级总GMV规模影响       0
父级总体毛利率影响     -10
---------------------------
父级毛利变化           -10
```

这才回答：

> “父级毛利为什么下降，是总规模还是总体毛利率？”

### 2.3 Parent Margin Mix / Rate

再对总体毛利率 `20% → 15%` 做 Mix / Rate：

```text
Mix Effect（结构效应）   -5.0pp
Rate Effect（单元率效应） 0.0pp
-----------------------------
总体毛利率变化            -5.0pp
```

于是整条经营解释变成：

> 父级总 GMV 没有变化；毛利下降全部来自总体毛利率下降，而总体毛利率下降又全部来自 GMV 结构从高毛利单元向低毛利单元迁移。

该结论符合业务直觉。

---

## 3. 第二轮后的正式路由规则

### 3.1 问“谁拖累 / 拉动了毛利”

例如：

> 哪些门店×渠道拖累湖南毛利？

使用：

> **Roll-up Attribution View（原子归因向上聚合）**

因为问题关心的是原子业务单元的影响清单。

允许按：

- 门店；
- 门店类型；
- 渠道；
- 门店×渠道；
- 未来分部 / 大区；

对同一 Parent Context 下的原子影响做 SUM。

### 3.2 问“为什么这个对象自己的毛利变化”

例如：

> 为什么湖南毛利下降？是 GMV 还是毛利率？

先把湖南重新作为 Parent，聚合湖南本期 / 基期的：

```text
总GMV
总毛利
总体毛利率
```

然后使用：

> **Parent Re-decomposition View（父级重新分解视图）**

做 Bennett：

- Parent Total GMV Scale Effect（父级总 GMV 规模影响）；
- Parent Gross Margin Rate Effect（父级总体毛利率影响）。

### 3.3 问“为什么这个对象自己的毛利率变化”

例如：

> 为什么湖南毛利率下降？

使用：

> **Parent Mix / Rate Re-decomposition（父级毛利率重新做结构 / 单元率分解）**

原子仍使用 `门店 × 渠道`，但权重必须相对于当前 Parent 重新计算。

---

## 4. 纯规模测试

构造：

| 原子 | 基期GMV | 本期GMV | 毛利率 |
|---|---:|---:|---:|
| A | 100 | 120 | 30% |
| B | 100 | 120 | 10% |

结果：

```text
父级GMV：200 → 240
父级毛利：40 → 48
父级毛利率：20% → 20%
```

Atomic Roll-up：

```text
GMV变动影响 +8
率变动影响    0
```

Parent Re-decomposition：

```text
父级总GMV规模影响 +8
总体毛利率影响      0
```

Mix / Rate：

```text
Mix  0
Rate 0
```

两种 View 完全一致。

判定：**PASS**。

---

## 5. 纯率变化测试

总 GMV 与结构保持不变，只改变原子毛利率。

例如两个原子 GMV 都保持 100：

```text
A：30% → 35%
B：10% → 15%
```

则：

```text
父级GMV：200 → 200
父级毛利：40 → 50
总体毛利率：20% → 25%
```

Atomic Bennett：

```text
GMV变动影响    0
率变动影响   +10
```

Parent Bennett：

```text
父级总GMV规模影响   0
总体毛利率影响    +10
```

Parent Mix / Rate：

```text
Mix Effect  0
Rate Effect +5.0pp
```

判定：**PASS**。

---

## 6. Time Reversal（时间反转）测试

Symmetric Bennett 必须满足：

> 如果把“基期 → 本期”反过来分析成“本期 → 基期”，各影响值只应符号反转，绝对值不应改变。

第一轮极端场景复测全部满足：

```text
Effect(1 → 0) = -Effect(0 → 1)
```

这说明算法不存在基期 / 本期的方向偏见。

判定：**PASS**。

> **Historical note：当前 Baseline 已进一步明确，Entry / Exit 在时间反转时需要角色互换后反号，不能把本节的“同名反号”推广到所有组件。**

---

## 7. Roll-up Compatibility（向上聚合兼容性）重新定义

第一轮的“可向上聚合”需要在第二轮收紧定义。

### 7.1 可以安全 SUM 的内容

在同一 Parent Context、同一比较期间、同一算法版本下：

- 原子 GMV 变化额；
- 原子毛利变化额；
- 原子 Bennett 的 GMV 变动影响；
- 原子 Bennett 的毛利率变动影响；
- Parent Mix / Rate 产生的原子贡献；
- Shadow 的 Base / Interaction 贡献；

均可以按门店、门店类型、渠道等进行 SUM。

### 7.2 SUM 后不能擅自改名

例如：

```text
Σ Atomic Scale
```

只能称：

> 原子 GMV 变动影响的 Roll-up 合计。

不能自动改称：

> 父级总 GMV 规模影响。

后者必须把父级对象重新聚合后做 Parent Bennett。

### 7.3 必须绑定 View Type

建议所有分解结果至少带：

```text
view_type = ATOMIC_ROLLUP | PARENT_REDECOMPOSITION
parent_context
comparison_period
algorithm_version
```

避免 AI 将两个合法但不同的问题混成一个结论。

---

## 8. Atomic Closure 与 Parent Closure

第二轮正式区分两个闭合概念。

### Atomic Closure（原子闭合）

对每一个合法原子：

```text
Atomic Scale_i + Atomic Rate_i = Δ毛利_i
```

这是第一轮 Bennett 的硬约束。

### Parent Closure（父级闭合）

当前 Parent 重新聚合后：

```text
Parent Scale + Parent Rate = Δ毛利_parent
```

以及：

```text
Σ Mix_i + Σ Rate_i = Δ总体毛利率_parent
```

两类闭合都必须成立，但两套 Scale / Rate 不要求数值相同。

---

## 9. Entry / Exit（新进入 / 退出）第二轮修正

当原子一侧 GMV = 0 时，该侧毛利率不存在。

因此不能：

- 静默把毛利率填成 0%；
- 使用普通 Bennett；
- 使用普通对称 Mix / Rate 强行分配。

### 9.1 毛利额 Impact Ledger

新进入：

```text
Entry GP Effect = 本期毛利额
```

退出：

```text
Exit GP Effect = -基期毛利额
```

不强行拆成 Scale / Rate。

> **Historical note：当前 Baseline 已进一步要求 Pure Entry / Exit 必须由 `ABSENT` Key Presence 判定，而不能只看净 GMV=0。**

### 9.2 毛利率 Parent Decomposition

对于新进入单元：

```text
Entry Rate Contribution_i = w1_i × r1_i
```

对于退出单元：

```text
Exit Rate Contribution_i = -w0_i × r0_i
```

现存单元继续使用普通 Mix / Rate。

这样：

> 不需要给不存在的期间虚构毛利率，同时仍可以严格闭合总体毛利率变化。

因此第二轮建议：

> **Entry / Exit 独立列示，不默认塞进 Mix。**

判定：第一轮边界规则得到收紧。

> **DEPRECATED：本节 9.2 的 Entry / Exit 毛利率直接贡献公式已在第三轮被判 Semantic FAIL，不得用于当前实现。**

---

## 10. GMV = 0 但毛利 ≠ 0

如果：

```text
GMV = 0
毛利 ≠ 0
```

则：

```text
毛利 = GMV × 毛利率
```

在该原子已经失效。

可能来自返利调整、冲销、历史结算等业务记录，但 A 阶段不得自行推测原因。

处理：

> **Non-standard / Adjustment Atom（非标准 / 调整型原子）**

要求：

- 保留原始金额；
- 进入 GMV / 毛利金额台账；
- 不进入普通 Bennett 或 Mix / Rate；
- 不删除、不填假毛利率、不为了闭合静默修正。

判定：**PASS**。

> **Historical note：当前 Data Contract 已将底层 Machine Code 改为 `ZERO_GMV_NONZERO_GP`，避免把数值状态直接命名成现实“调整”。**

---

## 11. Negative GMV（负 GMV）与极小分母

### 11.1 Negative GMV

Bennett 代数上仍可能计算，但毛利率权重 `w_i` 会失去通常“份额”的经营直觉。

V0.1 建议：

> 原子 GMV < 0 时，不静默进入普通 Mix / Rate；先标记为 Non-standard / Adjustment Candidate，再结合真实数据确认业务语义。

不因为公式“算得出来”就强行解释。

### 11.2 Tiny Denominator（极小 GMV 分母）

当 GMV 非零但极小时，毛利率可能出现巨大波动。

V0.1 暂不写死任意金额阈值，也不 winsorize（缩尾）或删行。

要求：

- 保留原始值；
- 增加 denominator-risk / small-base WARN；
- 后续用真实月度数据决定是否需要稳定阈值。

---

## 12. YTD（累计）复测

金额：

```text
YTD GMV = Σ月度GMV
YTD 毛利 = Σ月度毛利
```

可以直接向上聚合。

但 Mix / Rate 不得把每月结果直接相加冒充 YTD WHY。

正确流程：

```text
月度原子状态
↓
按门店×渠道聚合出YTD基期 / 本期状态
↓
重新计算YTD毛利率和权重
↓
重新运行YTD Mix / Rate
```

原因：

> Parent Context 与权重已经变化。

判定：**PASS**。

---

## 13. Shadow V1 第二轮增强：不仅看 Interaction，还要看 Offset

第一轮 Shadow 已定义：

### Interaction Intensity（交互强度）

```text
|Interaction|
/
(|Base Scale| + |Base Rate| + |Interaction|)
```

它回答：

> GMV 与毛利率同时变化的联合部分有多强？Production 的两因素分配对“变化顺序”有多敏感？

第二轮新增：

### Offset Intensity（对冲强度）

Production Bennett 已得到：

```text
Scale
Rate
Δ毛利 = Scale + Rate
```

定义：

```text
Offset Intensity
= 1 - |Δ毛利| / (|Scale| + |Rate|)
```

当分母为 0 时记为 0 / N.A.。

范围理论上为 `0% ~ 100%`。

含义：

- 接近 0%：两个因素同方向或几乎没有对冲；
- 接近 100%：两个因素大幅反向，净额掩盖了大量内部变化。

例如：

```text
GMV规模拉动 +68
毛利率拖累   -72
净毛利变化    -4
```

则：

```text
Offset Intensity
= 1 - 4 / (68 + 72)
≈ 97.1%
```

这类单元即使净毛利变化很小，也应该被 Shadow Attention 捕捉。

> **Historical note：当前 Baseline 已将 Offset 升级为 Production Kernel，并正式规定 Gross=0 时 Offset=N/A。**

---

## 14. Shadow V1 的两个不同提升点

### 14.1 Interaction Risk（交互风险）

由显式 Interaction decomposition 发现：

> GMV 与毛利率同时变化的联合项很大，Production 虽然公允，但“主要是谁”的表达需要克制。

主要影响：

- 结论措辞；
- 归因置信度；
- 是否需要进一步解释量率共同变化。

### 14.2 Offset Risk（对冲风险）

由 Offset Intensity 发现：

> 净结果很小，但内部存在很大的正负拉动 / 拖累。

主要影响：

- Attention 排序；
- 防止只按净变化绝对额漏掉重要单元；
- 识别“表面稳定、内部剧烈变化”的对象。

因此：

> **Interaction 与 Offset 不是同一件事。一个衡量联合变化，一个衡量反向抵消。**

---

## 15. 毛利率 Shadow 的对应形式

Production：

```text
Mix_i  = Δw_i × (r0_i + r1_i) / 2
Rate_i = Δr_i × (w0_i + w1_i) / 2
```

Shadow 显式保留：

```text
Base Mix_i    = Δw_i × r0_i
Base Rate_i   = w0_i × Δr_i
Interaction_i = Δw_i × Δr_i
```

满足：

```text
Base Mix_i + Base Rate_i + Interaction_i
= w1_i r1_i - w0_i r0_i
```

且 Production 等价于：

```text
Mix_i  = Base Mix_i  + 1/2 × Interaction_i
Rate_i = Base Rate_i + 1/2 × Interaction_i
```

因此 Shadow 可以用同一套 Interaction Intensity 思想监测毛利率结构 / 单元率解释的稳定程度。

---

## 16. 第二轮公允性结论

第二轮后，“公允”需要增加第九条：

> **Semantic Scope Consistency（语义范围一致）：算法结果必须绑定其 Parent Context 与 View Type，不能把 Atomic Roll-up 的因素名冒充 Parent Re-decomposition 的因素名。**

完整评审：

| 要求 | 结果 |
|---|---|
| Closure（闭合） | PASS |
| Order Independence（顺序无关） | PASS |
| Symmetry（对称） | PASS |
| Time Reversal（时间反转） | PASS |
| Atomic Reproducibility（原子可复算） | PASS |
| Same-context Roll-up（同父级可聚合） | PASS |
| Parent Re-decomposition（换父级重算） | PASS，必须执行 |
| Low Explanation Cost（低解释成本） | PASS，但必须区分两个 View |
| Semantic Scope Consistency（语义范围一致） | PASS，新增硬约束 |

结论：

> **第一轮 Production 公式不需要推翻，但其 Roll-up 语义必须收紧。**

---

## 17. 第二轮 Production 正式状态

### GMV

```text
Production：原子 ΔGMV
```

回答 WHERE。

### 毛利额 — Atomic Roll-up View

```text
Production：Atomic Symmetric Bennett
├─ Atomic GMV Change Effect
└─ Atomic Gross Margin Rate Change Effect
```

要求每行闭合到原子毛利变化，可用于影响台账、排序和向上 SUM。

### 毛利额 — Parent Re-decomposition View

```text
Parent 毛利变化
├─ Parent Total GMV Scale Effect
└─ Parent Gross Margin Rate Effect
```

回答“这个对象自己为什么变化”。

### 毛利率 — Parent WHY

```text
总体毛利率变化
├─ Mix Effect（结构效应）
├─ Rate Effect（单元自身毛利率效应）
├─ Entry Effect（新进入，若有）
└─ Exit Effect（退出，若有）
```

全部绑定当前 Parent Context。

> **Historical snapshot only：本节反映第二轮当时状态，当前 Production 以最新 Baseline 为准。**

---

## 18. 第二轮 Shadow 正式状态

```text
Shadow V1
├─ Explicit Interaction Decomposition
│  └─ Interaction Intensity
└─ Offset Detection
   └─ Offset Intensity
```

Shadow 的目标不是替代 Production，而是测试是否：

1. 找到 Production Attention 漏掉的单元；
2. 改变优先级；
3. 改变结论语言强度；
4. 改变 Evidence 搜集方向。

若连续真实月份没有产生上述增量价值，则删除或降级 Shadow。

> **Historical note：Offset 后续已升级为 Production。**

---

## 19. 第二轮最终判断

**Production：PASS，但新增 View Type / Parent Context 硬约束。**

**Shadow V1：PASS，第二轮新增 Offset Intensity，增量价值比第一轮更明确。**

最重要的第二轮结论不是新公式，而是：

> **同一个数学结果只有在问题语义、Parent Context 和 View Type 一致时才是“公允”的。原子归因可以安全向上聚合，但不能因此冒充父级对象自己的重新分解。**

这与既有 Roll-up Engine 原则完全一致：

> **Roll-up View 与 Re-decomposition View 必须区分。**

> **当前实施规则若与本历史文件冲突，以当前 Production Baseline / Data Contract 为准。**

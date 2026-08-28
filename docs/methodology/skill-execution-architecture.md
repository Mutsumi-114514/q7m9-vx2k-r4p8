# Skill Execution Architecture：AI、代码与穷举计算的职责分工

> 定位：把 A 阶段方法论真正落成 Skill 时的执行架构。  
> 目标：明确 AI 不应该承担什么、确定性代码必须承担什么，以及“穷举”如何成为系统默认执行方式。

---

## 1. Skill 不是为了教 AI 算 `SUM(A1:A2)`

基础算术、SUM、Join、Unpivot、Bennett、Mix / Rate、Bridge、Offset 等一旦定义清楚，就应该尽量交给确定性代码执行。

Skill 真正值得存在的是：

- 什么时候 SUM；
- 什么绝对不能 SUM；
- SUM 后语义是否还能继承；
- Parent / View / Scope 改变后什么必须重算；
- 当前原子处于什么 Period State / Transition；
- 哪套算法路径合法；
- 哪些结果允许解释到 Mathematical WHY，哪些必须停止；
- 下一步是否继续下钻或进入 Evidence。

核心原则：

> **让代码负责确定性计算，让 Schema / Type 负责约束，让 AI 负责理解、路由和解释。**

---

## 2. AI 的两个阶段角色

### Build-time AI（构建期 AI）

```text
人提出经营判断 / 边界
↓
AI帮助形式化
↓
生成公式 / 代码 / 反例测试
↓
人审查业务语义
↓
冻结为确定性函数与协议
```

AI 在这里可以“写代码”。

### Run-time AI（运行期 AI）

正式运行后，AI 不应该每个月重新心算公式。

运行期角色：

- **Schema Interpreter（结构解释器）**：识别宽表 / 长表，发现陌生字段时请求确认；
- **Parser（问题解析器）**：解析 Metric / Scope / Comparison / Question Type；
- **Router（分析路由器）**：选择 Atomic Attribution、Parent Re-decomposition、Continuing Mix / Rate、Bridge 等路径；
- **Controller（执行控制器）**：检查 Parent Context、Boundary、Closure、Type、Invariant；
- **Interpreter（结果解释器）**：把结构化数学结果翻译成经营语言，并遵守 Fact / Inference / Evidence 边界。

AI 不是主要的：

> **Calculator（计算器）。**

---

## 3. Skill 的四层结构

```text
Skill
│
├─ ① Calculation Kernel（确定性计算内核）
│    SUM / Join / Unpivot
│    Presence / State Classification
│    Bennett
│    Continuing Mix-Rate
│    Entry-Exit / Non-standard Bridge
│    Gross Movement / Offset
│
├─ ② Semantic Contract（语义合同）
│    什么能SUM
│    什么必须重算
│    Parent是谁
│    View是什么
│    State / Transition是什么
│    哪些值禁止解释
│
├─ ③ Decision Protocol（决策协议）
│    WHAT / WHERE / WHY？
│    Atomic Attribution 还是 Parent Re-decomposition？
│    当前应该调用哪套确定性函数？
│
└─ ④ Interpretation Layer（AI解释层）
     把结构化结果翻译成人话
     形成 Mathematical WHY
     必要时进入 Hypothesis / Evidence
```

工程原则：

> **Code enforces（代码负责强制）**  
> **Schema constrains（结构负责约束）**  
> **Prompt explains（提示词负责解释）**

错误路径应尽量由系统结构直接禁止，而不是只靠“请 AI 不要犯错”。

---

## 4. Exhaustive Calculation, Selective Attention

当前 A 阶段核心执行原则：

> **Exhaustive Calculation, Selective Attention（计算穷举，注意力筛选）。**

不要先选 Top N 再算，而应该先把全部合法原子计算完，再决定展示哪些。

例如：

```text
GMV规模拉动      +300
毛利率变化拖累    -304
净毛利变化          -4
```

只按净变化排序可能完全看不到，但 Gross Movement / Offset 会暴露其内部剧烈运动。

因此：

> **计算层永远穷举；展示层才做 Attention。**

---

## 5. 输入层保持最小复杂度

内部表格格式整体稳定，因此不设计万能“脏表 AI 清洗器”。

Gate 0 只做：

```text
输入 CSV / Excel
↓
识别 Wide Table（宽表）
或 Long / Unpivot Table（长表 / 逆透视表）
↓
字段词典匹配
↓
存在词典外字段？
├─ 否 → 继续
└─ 是 → 询问用户确认
↓
Canonical Input（标准输入）
```

例如发现：

```text
常规NETGMV
```

只需确认：

> 是否对应标准字段“常规GMV”？

原则：

> **现实本来简单的地方不要过度工程化；可能造成业务语义错误的地方才需要强约束。**

---

## 6. Atomic Ledger 应一次性穷举

对于比较期 Base 与 Current，不构造理论笛卡尔积，而取真实出现过的原子坐标并集：

```text
Atoms = Keys_Base ∪ Keys_Current
```

使用 Full Outer Join 保留 Entry / Exit。

关键纪律：

> **Full Outer Join 后必须先保存 Presence，再处理金额 Null。**

至少保存：

```text
base_key_present
current_key_present
```

禁止先把所有 Null 填 0，再用 `GMV=0` 推断“无业务”，否则会把 `ABSENT` 与 `NET_ZERO_PRESENT` 混为一谈。

对每个 `时间 × 门店 × 渠道` 原子，确定性计算层至少生成：

```text
基期GMV / 本期GMV / ΔGMV
基期毛利 / 本期毛利 / Δ毛利
基期毛利率 / 本期毛利率
base_key_present
current_key_present
base_state
current_state
transition_type
boundary_flags
atomic_gp_effect_total
```

### 6.1 Period State 使用固定 Machine Code + 中文标准名

Period State 以数据合同为唯一 Source of Truth，至少包括：

| Machine Code | 中文标准名 |
|---|---|
| `ABSENT` | 无原子记录 |
| `STANDARD` | 标准业务状态 |
| `NET_ZERO_PRESENT` | 有记录但净额归零 |
| `ZERO_GMV_NONZERO_GP` | 零GMV非零毛利 |
| `NEGATIVE_GMV` | 负GMV状态 |
| `INVALID_OR_MISSING` | 数据缺失或无效 |
| `OTHER_NONSTANDARD` | 其他非标准状态 |

AI 不得自行翻译或改名。

### 6.2 Transition Type

至少包括：

| Machine Code | 中文标准名 |
|---|---|
| `CONTINUING_STANDARD` | 标准存量业务 |
| `PURE_ENTRY` | 纯新增业务 |
| `PURE_EXIT` | 纯退出业务 |
| `NONSTANDARD_TRANSITION` | 非标准状态迁移 |
| `FULL_MEMBERSHIP_REPLACEMENT` | 标准业务集合完全替换 |
| `BOUNDARY_STOP` | 边界终止 |

其中只有：

```text
ABSENT → STANDARD
```

才能自动判 `PURE_ENTRY`；只有：

```text
STANDARD → ABSENT
```

才能自动判 `PURE_EXIT`。

### 6.3 可提前物化的结果

Parent-independent 结果可以物化：

- ΔGMV / Δ毛利；
- `atomic_gp_effect_total = current_gp - base_gp`；
- `CONTINUING_STANDARD` 的 Atomic Bennett；
- Pure Entry / Exit / Non-standard 的原子金额组件；
- 原子级 Bennett Gross Movement / Factor Offset；
- 原子 Boundary Flags。

统一闭合：

```text
Σ atomic_gp_effect_total = Δ Parent GP
```

---

## 7. Parent-dependent 指标不能提前写死

以下结果依赖 Parent Context：

- GMV 权重；
- Parent Bennett；
- Continuing Mix / Rate；
- Entry / Exit Rate Effect；
- Non-standard Bridge；
- Contribution %；
- Parent / Unit Offset。

因此它们应由 Contextual Engine 在 Parent 确定后重新穷举。

整体结构：

```text
Exhaustive Atomic Ledger
        │
        ├─ Atomic Query / Attribution Roll-up
        │
        └─ Contextual Engine
             ├─ Parent Bennett + Preconditions
             ├─ Total → Standard Non-standard Bridge
             ├─ Standard → Continuing Entry / Exit Bridge
             ├─ Continuing Mix / Rate
             ├─ Full Membership Replacement Boundary
             ├─ Contribution
             └─ Context-specific Offset
```

### 7.1 Parent Bennett 必须有 Gate

只有：

```text
Parent G0 != 0
Parent G1 != 0
Parent r0 / r1 合法
Parent 数据状态通过校验
```

才运行 Parent Bennett。

否则返回：

```text
view_result = N/A
boundary_status = BOUNDARY_STOP
boundary_reason = <明确原因>
```

---

## 8. 毛利率 Contextual Engine 的状态层级

为了避免非标准业务与 Entry / Exit 重复计算，Parent Rate 必须分三层：

```text
Total Parent
↓ 去除 Non-standard
Standard Parent
↓ 去除 Entry / Exit
Continuing Standard Parent
```

普通 Mix / Rate 只对 Continuing Standard Set 运行，并在该集合内部重新归一化权重。

这样代码路径本身就能阻止：

- 给 GMV=0 的原子制造假毛利率；
- 把 Negative GMV 当普通 Mix；
- 把 `NET_ZERO_PRESENT` 当成“无业务”；
- Non-standard 重复计入 Entry；
- 使用 Total Parent 权重分解 Continuing Rate。

---

## 9. 结构化结果必须有 Type + Unit + Version

一个结果不能只是：

```text
-10
```

至少应知道：

```text
metric
unit_of_measure
grain
scope
comparison_period
view_type
parent_context
decomposition_type
component_set
base_state/current_state 或 transition_type
algorithm_version
data_contract_version
metric_definition_version
input_fingerprint 或 source_data_version
boundary_status
boundary_reason
closure_error
```

### 9.1 Machine Code / 中文标准名 / 展示语言分层

所有重要枚举至少维护：

```text
machine_code
zh_cn_label
executable_definition
display_guidance
```

例如：

```text
code = NET_ZERO_PRESENT
zh_cn_label = 有记录但净额归零
```

AI 只负责在合同允许范围内组织自然语言，不负责重新发明标准术语。

### 9.2 Unit 必须结构化

建议至少：

| Machine Code | 中文标准名 |
|---|---|
| `CNY_10K` | 万元 |
| `RATE` | 比率 |
| `PERCENTAGE_POINT` | 百分点 |
| `COUNT` | 数量 |

例如 Atomic Bennett Rate Effect 的数值单位可能是 `CNY_10K`，而 Continuing Rate Effect 的单位是 Rate / Percentage Point 语义；不能因为名字里都有 `Rate Effect` 就混用。

---

## 10. Executable Invariants（可执行不变量）

不变量不应该只写在 Prompt 里，而应由程序自动断言。

至少包括：

```text
Atomic GP Total Closure
Atomic Bennett Closure
Parent Bennett Closure
Parent Rate Bridge Closure
Continuing Mix / Rate Closure
Time Reversal with Component Mapping
Derived Ratio No-Roll-up
Boundary Routing
Presence / Zero Separation
Unrounded Closure
```

并按 Algorithm Applicability Matrix 只执行适用的不变量。

任一失败：

> 不允许进入 AI Interpretation。

---

## 11. 运行期推荐链路

```text
Raw Table
↓
Format Detection + Unknown Field Confirmation
↓
Canonical Input
↓
Build Base / Current Key Presence
↓
Build Exhaustive Atomic Ledger
↓
Classify Period States / Transitions
↓
Validate Data Contract + Atomic Closure
↓
AI Parser / Router
↓
Deterministic Contextual Functions
↓
Validate Contextual Invariants
↓
Structured Result with Type / Unit / Scope / View
↓
AI Interpreter
↓
Mathematical WHY
↓
必要时进入 Hypothesis → Evidence
```

AI真正开始自由组织语言时，前面的数学计算应已经结束并通过校验。

---

## 12. 高手经验显化是核心价值

机器执行的难点不在算术，而在人脑默认拥有大量未写下的语义先验。

例如人看到：

```text
高毛利业务GMV下降
低毛利业务GMV上升
Parent总GMV不变
```

会立即判断：

> 结构恶化。

机器如果只被告诉“原子 Bennett 后 SUM”，可能在数字正确时把它错误命名为 Parent GMV Scale Effect。

又例如：

```text
两条合法记录正负抵消后 GMV=0、毛利=0
```

人可能知道“发生过业务”；机器若只看净值，可能误判为“此前不存在”。

因此 Skill 的核心工作是：

> **Tacit Knowledge（隐性经验） → Explicit Rule（显式规则） → Executable Constraint（可执行约束）。**

---

## 13. 当前学科基础

当前体系位于多门成熟学科的交叉处：

- **Index Number Theory（指数理论）**：Time Reversal、公允分解、Bennett / Divisia 传统；
- **Management Accounting Variance Analysis（管理会计差异分析）**：Price / Volume / Mix；
- **OLAP / Dimensional Modeling（多维分析 / 维度建模）**：Atomic Grain、Roll-up、Additive / Non-additive Measure；
- **Formal Methods / Design by Contract（形式化方法 / 契约式设计）**：Precondition、Postcondition、Invariant；
- **Measurement Theory（测量 / 语义理论）**：数学可运算不等于语义可继承；
- **Cooperative Game Theory / Interpretable ML（合作博弈论 / 可解释机器学习）**：Shapley 等公平分配交互思想；
- **Causal Inference（因果推断）**：Attribution ≠ Causation。

---

## 14. 当前总原则

> **机器不需要像人一样“顿悟”，但系统必须把人的顿悟转成机器难以违反的规则。**

最终目标不是一个会背经营分析方法论的聊天机器人，而是一套：

> **确定性计算穷举 + 显式类型 / 语义合同 + AI 路由与解释 + Evidence 边界。**

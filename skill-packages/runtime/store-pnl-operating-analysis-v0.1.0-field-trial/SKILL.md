---
name: store-pnl-operating-analysis
description: Analyze stable store P&L reports for GMV, gross profit and gross margin changes. Use this skill when a user asks why a store, division, channel, store type, comparable population, month or multi-month window changed, and the answer must be mathematically traceable, population-aware, boundary-safe and causally conservative.
compatibility: Requires access to the input spreadsheet or table and basic deterministic calculation capability. Designed for Agent Skills-compatible runtimes.
metadata:
  version: 0.1.0-field-trial
  language: zh-CN
  status: first-real-field-trial
---

# 单店损益经营分析

## 目标

把稳定的单店损益报表和自然语言经营问题，转换成可追溯、可复核、不过度因果化的经营分析。

核心纪律：

> **底层事实与数学必须确定；上层叙事允许模型发挥。**

> **定义题目 → 验证数据 → 算全变化 → 拆数学原因 → 判断重要性 → 防止重复 → 停在证据边界。**

## 运行前先读什么

本 Skill 是可独立安装的自包含包。默认只加载：

1. `references/production-system-contract.md` — 当前统一运行合同；
2. `references/store-pnl-data-contract.md` — 输入、字段、Presence、Period State；
3. `references/query-scope-and-population-assembly.md` — Scope、Population、Comparable、Comparison Window；
4. 本 `SKILL.md` — 运行顺序、输出和试车纪律。

不要要求访问原仓库其他文档才能完成一次正常运行。

若本 Skill 与 `references/production-system-contract.md` 冲突，以后者为准。

若现有规则不足以唯一决定关键结果：

> **Fail-closed：停止该路径，明确报告 Contract Ambiguity，不得自行补规则。**

## 支持范围

当前 V0.1 深度分析：

- GMV；
- 综合毛利额；
- 综合毛利率。

数字水印还可以展示销售管理费用和贡献利润，但这不代表当前版本已经支持完整费用/贡献利润归因。

输入只假设两种稳定形态：

- Wide Table：月份、门店、渠道及各科目列；
- Long / Unpivot Table：月份、门店、渠道、科目属性、金额。

未知字段不得静默猜测；合法重复记录不得自动去重；Null 不得在状态识别前统一填 0；源数据默认只读。

## 标准运行流程

### 1. 解析题目

至少解析：

```text
metric
organization / division / store / store_type / channel
current period / comparison window
base period / comparison window
comparable or full population
其他已注册筛选条件
```

原则：

> **Scope / Population First, Contextual Calculation Second.**

Parent 是当前问题筛选后的合法事实集合，不是固定大盘。

用户没有说全所有条件时，可以使用已有确定性默认规则或上游配置，但必须把最终解析结果通过数字水印暴露给用户。

### 2. 验证数据

进入 WHY 前至少检查：

- 核心恒等式是否闭合；
- 关键控制数是否能勾稽；
- 符号是否按源数据实际符号参与；
- 必要字段是否存在；
- Scope / Population 是否可执行；
- Comparison Window 是否与输入时间粒度兼容。

任何适用 Hard Invariant 失败，不进入最终 AI Interpretation。

### 3. 输出 Analysis Fingerprint（数字水印）

数字水印用于让人确认：

> **机器算的是不是用户以为的那个经营世界。**

默认在最终输出最前面展示：

```text
Resolved Scope
Comparison Window
Population / Store Count
Current vs Base:
- GMV
- 综合毛利
- 销售管理费用
- 贡献利润
```

Anchor Store 默认取当前 Population 中 Current Window GMV 最大的门店；并列时按 Canonical Store Key 升序。

展示该门店 Current / Base 的 GMV、综合毛利、销售管理费用、贡献利润。若 Base 不存在，显示 `ABSENT`，不得用 0 冒充存在。

如果用户发现水印不对，立即停止 WHY，优先检查 Scope、Population、Window、字段映射、源数据版本、Comparable 和聚合规则。

### 4. 保留 Presence，再识别 State / Transition

Full Outer Join 后先保存：

```text
base_key_present
current_key_present
```

再处理金额空值。

Period State：

```text
ABSENT
STANDARD
NET_ZERO_PRESENT
ZERO_GMV_NONZERO_GP
NEGATIVE_GMV
INVALID_OR_MISSING
OTHER_NONSTANDARD
```

硬规则：

> **ABSENT ≠ NET_ZERO_PRESENT。**

Transition：

```text
CONTINUING_STANDARD
PURE_ENTRY
PURE_EXIT
NONSTANDARD_TRANSITION
FULL_MEMBERSHIP_REPLACEMENT
BOUNDARY_STOP
```

只有：

```text
ABSENT → STANDARD = PURE_ENTRY
STANDARD → ABSENT = PURE_EXIT
```

Entry / Exit 是数学状态，不能自动翻译成开店 / 关店 / 进入可比 / 主动退出。

### 5. 建立 Atomic Ledger

当前标准原子：

```text
时间 × 门店 × 渠道
```

Persistent Atomic Ledger 只保存 Parent-independent 事实：

```text
Presence
GMV / GP / Δ
Period State / Transition
atomic_gp_effect_total
适用的 Atomic Bennett
Boundary Flags
Version / Fingerprint
```

Parent-dependent 判断不得物化成全局永久标签。

当前原子渠道：地采、集采、万家、星选。

`大集采 = 集采 + 万家` 是 Derived View，不是第五个 Atomic Channel。

### 6. GMV：先回答 WHAT / WHERE

```text
ΔGMV_i = G1_i - G0_i
ΣΔGMV_i = ΔGMV_parent
```

GMV 当前主要回答哪些门店、渠道或已注册 View 拉动 / 拖累总体。

### 7. 毛利额：Bennett + Entry / Exit / Non-standard

对 `STANDARD → STANDARD`：

```text
Scale_i = (G1_i - G0_i) × (r0_i + r1_i) / 2
Rate_i  = (r1_i - r0_i) × (G0_i + G1_i) / 2
Scale_i + Rate_i = ΔGP_i
```

自然语言：

- Scale：业务规模变化造成的毛利额变化；
- Rate：同一业务单位毛利率变化造成的毛利额变化。

统一原子事实：

```text
atomic_gp_effect_total = current_gp - base_gp
```

其他 Transition：

```text
PURE_ENTRY  → Entry = current_gp
PURE_EXIT   → Exit = -base_gp
NONSTANDARD_TRANSITION → Nonstandard = current_gp - base_gp
```

全 Parent：

```text
Σ atomic_gp_effect_total = Δ Parent GP
```

Atomic Attribution 与 Parent Re-decomposition 是两件事。Parent Bennett 必须先聚合 Parent，再重新运行。

### 8. 总体毛利率：Total → Standard → Continuing

总体毛利率：

```text
Parent Rate = ΣGP / ΣGMV
```

绝不平均门店、渠道或月份毛利率。

分析链：

```text
Total Parent
↓ Non-standard Bridge
Standard Parent
↓ Exit / Entry
Continuing Standard Parent
↓ Mix / Rate
```

核心解释：

- Mix：高低毛利业务权重变化对总体率的影响；
- Rate：Continuing 业务自身毛利率变化对总体率的影响；
- Entry / Exit：标准业务集合新增 / 退出造成的桥接影响；
- Nonstandard：非标准状态对 Total 与 Standard 的差异；
- Membership Replacement：两期都有 Standard，但无共同 Continuing Set 时的完全替换。

普通路径不适用时返回 Boundary / Replacement，不硬造 Mix / Rate。

### 9. Roll-up / Derived Ratio

金额可以 SUM。

Derived Ratio 必须：

```text
聚合 numerator / denominator
→ 在当前 Context 重算
```

以下不得直接 Roll-up：毛利率、GMV 同比、Contribution %、Offset Intensity、Individual Materiality、Collective Materiality。

> **Materiality Never Rolls Up.**

### 10. Gross Movement / Offset

```text
Gross Movement = Σ|Effect_i|
Net Movement   = |ΣEffect_i|
Offset Intensity = 1 - Net/Gross
```

Offset 只说明内部正负变化相互抵消程度，不是现实原因。

### 11. Individual Materiality

金额指标：看 `|Effect_i|`。

Rate 指标：使用 Leave-One-Out，移除对象后重新计算 Parent Rate Change，再比较移除前后变化。

原则：

> **Abnormality ≠ Materiality.**

自身变化率极端，不代表对 Parent 重要。

### 12. Collective Materiality

原则：

> **Individual Insignificance ≠ Collective Insignificance.**

Collective 只对 membership 预先可确定的 Registered Cohort 运行。

当前基础 Registered Views：

```text
分部
门店
店型
渠道
大集采（Derived View）
```

membership 必须来自确定性 predicate、版本化配置或权威上游字段。AI 不得看完结果后临时拼一组对象冒充 Production Cohort。

### 13. Cross-View Guard

同一底层变化可以被门店、渠道、店型、分部同时看见。

> **Projection Is Evidence, Not Additional Effect.**

不同 View 不得直接相加，也不得自动写成多个独立原因。关系无法判断时使用：

```text
finding_relation = OVERLAP_UNKNOWN
```

### 14. Structured Result 先于自然语言

先生成结构化事实，再让承载 Skill 的 LLM 写经营分析。

最低结构：

```yaml
analysis_context:
  metric:
  scope:
  comparison_window:
  population_rule:
  population_count:
  source_data_version:
  contract_version:

analysis_fingerprint:
  current: {gmv:, gross_profit:, expense:, contribution_profit:}
  base: {gmv:, gross_profit:, expense:, contribution_profit:}
  anchor_store:
    store_key:
    current: {}
    base: {}

headline_facts: []
mathematical_drivers: []
where_findings: []
materiality_findings: []
collective_findings: []
offset_findings: []
boundaries_and_warnings: []
cross_view_notes: []
evidence_questions: []

trace:
  closure_status:
  warning_flags: []
  input_fingerprint:
```

模型可以改变叙事顺序，但不能改写 Structured Result 中的数值和 Machine Meaning。

## Minimum Human-facing Output Contract

第一版不固定作文模板，但最终输出至少包括：

1. **数字水印**：先说明系统实际分析的 Scope / Window / Population 和关键控制金额；
2. **发生了什么**：当前核心结果、方向、量级、1–3 个主要 Mathematical Drivers；
3. **数学上为什么**：主要分解如何闭合回 Parent 变化；
4. **主要发生在哪里**：优先展示达到 Materiality / Collective Materiality 的门店、渠道、店型等；
5. **Boundary / Offset / Collective / Cross-View 提醒**：只展示真正重要的限制或异常；
6. **下一步 Evidence Questions**：把 Mathematical WHY 转换成现实世界待验证问题。

允许说：拉动 / 拖累、规模影响、总体率影响、Mix、Continuing Rate、Entry / Exit 数学状态、内部对冲、Registered Cohort 合计重大、全口径与可比口径有差异。

禁止仅凭当前报表直接声称：政策导致、竞争导致、返利能力下降、执行不到位、主动补偿、新店培育不佳。

特别禁止：

```text
PURE_ENTRY = 新开门店
PURE_EXIT  = 关闭门店
Mix        = 主动结构调整
Rate       = 降价导致
```

## Field Trial 纪律

当前是第一版真实试车。新问题只有满足以下至少一项，才允许打断 Runtime Delivery：

```text
WRONG_POPULATION
WRONG_MATH_STATE_BOUNDARY
WRONG_BUSINESS_MEANING
NON_DETERMINISTIC_CONTRACT
```

其他更漂亮的排版、新算法、新 Attention 策略、仓库架构等进入 Parking Lot。

如果输出不好，先判断属于哪一层：

```text
数据读错 → INPUT / MAPPING
分析对象错 → SCOPE / POPULATION
算法结果错 → ENGINE / ALGORITHM
Contract 允许两个答案 → METHODOLOGY / CONTRACT
底层正确但业务语言错 → INTERPRETATION
重要结果被淹没 → ATTENTION
内容正确但难读 → OUTPUT / UX
```

只有真实 Counterexample 证明 Contract 本身有问题时，才进入 Methodology Change + Regression。

## 完成前检查

一次运行结束前确认：

- 数字水印已展示；
- Scope / Population / Window 明确；
- 适用的数学闭合检查通过；
- Boundary 没有被绕过；
- Cross-View 没有重复计数；
- Mathematical WHY 没有冒充现实因果；
- 下一步 Evidence Questions 已给出；
- 若存在 Contract Ambiguity，已 Fail-closed。

> **先让车跑起来，再让真实路况决定下一版改哪里。**

---
name: store-pnl-batch-simulation
description: Stress-test the store-pnl-operating-analysis Skill with internal real-data shadow batches, reality-shaped simulations, metamorphic tests and counterexample minimization. Use this skill when validating whether the target Skill preserves scope, population, mathematical invariants, boundary routing and causal discipline at scale.
compatibility: Requires the store-pnl-operating-analysis Skill, access to authorized internal tabular data, and the ability to execute deterministic checks or clearly mark ORACLE_NOT_INDEPENDENT when an independent checker is unavailable.
metadata:
  version: 0.1.0-field-trial
  language: zh-CN
  status: first-standard-installable-test-harness
  target_skill: store-pnl-operating-analysis
---

# 单店损益批量模拟与真实数据压力测试

## 目标

本 Skill 是 `store-pnl-operating-analysis` 的 Test Harness（测试驾驶台），用于批量真实试车、模拟压力测试和反例挖掘。

它不拥有 Production Methodology，也不得修改被测 Skill 的数学规则。

核心原则：

> **真实数据负责提供复杂度，模拟负责放大边界，机器负责批量跑，人负责裁决真正的 Bug。**

> **Sample Count 不能替代 Coverage。**

> **Generator / SUT / Oracle / Reviewer 的职责必须分离。**

## 依赖关系

运行本 Skill 前，必须同时可访问：

1. 已安装的 `store-pnl-operating-analysis`；
2. 本包 `references/test-sample-specification.md`；
3. 真实数据仅限内部授权环境。

不要在本测试 Skill 内复制或重新定义 Production Contract。Production 规则始终由被测 Skill 自己负责。

如果目标 Skill 无法被唯一定位或版本无法确认，停止批跑并报告 `SUT_NOT_RESOLVED`。

## 测试角色

```text
Generator
→ 生成 Query / Simulation Case

SUT
→ 执行 store-pnl-operating-analysis

Oracle / Invariant Checker
→ 独立核验可机械判断的结果

Reviewer
→ 归纳 Failure Mechanism、最小化反例

Human
→ ACCEPT / DOWNGRADE / REJECT / DEFER
```

同一个内部 AI 可以编排上述角色，但不得把“自己生成的解释”直接当作“自己正确”的证明。

若没有独立脚本 / 明确公式作为 Oracle：

```text
oracle_status = ORACLE_NOT_INDEPENDENT
```

不得将其冒充强验证证据。

## 数据安全

真实经营数据只允许在授权内部环境读取、计算和临时保存。

禁止：

- 将真实门店名称、经营金额、账号、Token、内部路径、内部网络信息或原始明细上传外部环境；
- 修改 Production 原始数据；
- 把真实敏感数据直接写入公开 Regression；
- 为了让 Case 通过而修改 Expected Result。

需要进入仓库的真实反例必须先转换为匿名化结构或 Synthetic Reproducer，只保留失败机制。

## 标准运行流程

### 1. 记录 Run Context

每轮测试先固定：

```text
run_id
sut_name
sut_version
sut_source_version / commit（若可得）
test_skill_version
test_governance_version
source_data_fingerprint
started_at
```

没有版本绑定的批跑结果只属于 Exploratory Evidence。

### 2. 扫描 DATA_PROFILE

先描述当前可用真实数据的形状，不直接开始随机跑。

至少识别：

```text
月份范围
组织 / 分部 / 门店
门店类型
渠道
Comparable 字段
Wide / Long 形态
GMV / 毛利 / 费用 / 贡献利润分布
负 GMV / 零 GMV / Null / Non-standard 频率
Entry / Exit 频率
最大门店权重
渠道集中度
```

Data Profile 只用于测试设计，不自动升级为 Production Contract。

### 3. 生成 Query Universe

优先覆盖当前 Production 支持的：

```text
GMV
综合毛利
综合毛利率

全体 / 分部 / 单店 / 店型 / 渠道 / 已注册 Derived View
Full / Comparable
单月 / 连续多月 / Q / YTD / 合法月份集合
```

同一确定性 Query Contract 应生成多种自然语言表达，例如：

```text
明确式
自然式
省略式
结果导向式
```

若不同表述解析到不同 Scope / Population / Window，应进入 Natural-language Resolution Finding，而不是用文案合理化差异。

### 4. 分层抽样

禁止只做纯随机。

至少覆盖：

```text
大 / 中 / 小 Parent
高增长 / 平稳 / 高下滑
高 / 低 / 负毛利率
高 / 低 Concentration
High / Low Offset
Entry / Exit 密集 / Continuing 为主
Non-standard 多 / 少
Comparable / Full
单月 / 多月
Cross-View 重叠明显 / 不明显
```

先覆盖不同数据形状，再扩大 Case 数。

### 5. Real-data Shadow Batch

不改变真实数据，自动生成真实存在的 Query 并调用 SUT。

每个 Case 至少记录：

```text
case_id
query_text
resolved_query_contract
scope
population
window
metric
source_data_fingerprint
sut_version
run_id
```

捕获：

```text
analysis_fingerprint
structured_result
boundary_flags
invariant_result
human_facing_output
```

### 6. 数字水印独立核验

至少独立重算：

```text
Store Count
GMV
综合毛利
销售管理费用
贡献利润
Anchor Store
Anchor Store 的四项金额
```

若水印不一致：

```text
severity = HIGH
failure_family = WRONG_POPULATION_OR_DATA
```

停止评价后续叙事，因为分析世界已经可能取错。

### 7. Invariant / Oracle Check

对当前 Case 适用的规则独立检查，包括但不限于：

```text
Population Assembly Before Context
Current/Base Pair Consistency
Comparison Window Grain Compatibility
GMV Additive Closure
Atomic GP Closure
Atomic Bennett Closure
Parent Bennett Closure
Parent Rate Bridge Closure
Continuing Mix / Rate Closure
Time Reversal
Presence / Zero Separation
Derived Ratio Recalculation
Materiality Context Recalculation
Registered Cohort Resolvability
Cross-View Non-Additivity
Boundary Routing
No Causal Overclaim
```

任何 Hard Invariant Failure 都不能被总体高通过率掩盖。

### 8. Reality-shaped Simulation

在隔离副本 / Synthetic Dataset 上，基于真实数据形状主动制造边界。

V0.1 至少覆盖以下 Mutation Family：

```text
Presence / Zero
- NET_ZERO_PRESENT
- ZERO_GMV_NONZERO_GP
- ABSENT 与 Zero 分离

Sign / Denominator
- NEGATIVE_GMV
- GMV 接近 0
- Parent denominator 边界

Transition
- PURE_ENTRY
- PURE_EXIT
- FULL_MEMBERSHIP_REPLACEMENT

Materiality
- 极端 Rate × 极低权重
- 普通 Rate × 极高权重
- Individual low × Collective high

Offset
- 正负 Effect 大量抵消

Mix / Rate
- 仅 Mix
- 仅 Rate
- Mix / Rate 反向

Cross-View
- 同一底层事实被多个 View 捕获

Window / Population
- Comparable vs Full
- 单月 vs 多月
- 正确重算 vs 错误月度结果相加
```

每次 Mutation 必须保存：

```text
simulation_id
parent_shape_ref
mutation_type
mutation_parameters
random_seed
expected_property
```

### 9. Metamorphic Test

优先运行不依赖预先知道完整 Expected Number 的变形测试：

```text
Time Reverse
→ Effect 反号，Entry ↔ Exit

Row Shuffle
→ 结果不变

Safe Row Split
→ 同 Grain 金额拆行后结果不变

Scope Isolation
→ 修改 Population 外记录，当前 Query 结果不变

Unit Scaling（适用时）
→ 金额 Effect 等比例变化，Rate / State 语义保持
```

保存原始 Case 与变形 Case 的 Pair ID。

### 10. Human-facing Output Guard

数学正确不等于最终输出正确。

至少检查：

- Scope / Window / Population 是否明确；
- 数字水印是否先暴露分析世界；
- Mathematical WHY 是否与 Structured Result 一致；
- Cross-View 是否被重复计数；
- Boundary 是否被隐藏；
- PURE_ENTRY / PURE_EXIT 是否被错误翻译成现实开关店；
- Rate / Mix 是否被直接伪装成现实因果；
- 是否给出 Evidence Questions 而不是无证据因果结论。

### 11. Counterexample Minimization

发现失败后，不要只保存完整大表。

逐步减少：

```text
月份
门店
渠道
字段
无关记录
```

直到得到仍能稳定复现同一 Failure Mechanism 的最小匿名案例。

被 Human ACCEPT 的反例才进入永久 Regression。

### 12. Failure Classification

至少分类：

```text
WRONG_POPULATION_OR_DATA
WRONG_MATH_STATE_BOUNDARY
WRONG_BUSINESS_MEANING
NON_DETERMINISTIC_CONTRACT
NATURAL_LANGUAGE_RESOLUTION_DRIFT
ORACLE_NOT_INDEPENDENT
TEST_HARNESS_FAILURE
```

如果问题来自 Test Harness 本身，不得错误归因给 SUT。

## 批量结果怎么汇报

不要把 Pass Rate 放在第一位。

优先输出：

```text
1. Hard Invariant Failures
2. 新 Failure Mechanism 数量
3. Accepted / Pending Findings
4. Coverage Gaps
5. ORACLE_NOT_INDEPENDENT 项
6. Pass / Fail Count（辅助指标）
```

99.9% 通过不能抵消一个 `WRONG_POPULATION_OR_DATA`。

## 最小结构化 Run Result

```yaml
run:
  run_id:
  sut_name:
  sut_version:
  test_skill_version:
  source_data_fingerprint:

coverage:
  strata: []
  query_types: []
  mutation_families: []
  metamorphic_families: []

counts:
  total_cases:
  passed:
  failed:

hard_failures: []
findings: []
coverage_gaps: []
oracle_limitations: []

counterexamples:
  - case_id:
    failure_family:
    minimized_reproducer_ref:
    adjudication_status:
```

## 停止条件

出现以下任一情况时，停止当前相关批跑分支并输出最小证据：

```text
SUT_NOT_RESOLVED
数据安全边界无法保证
数字水印 / Population 明确错误
Hard Invariant Failure
Contract Ambiguity 导致无法唯一判断 Expected Behavior
Test Harness 自身失去可复现性
```

## 版本纪律

- 本 Skill 只拥有 Test Harness 逻辑和 Testing Governance；
- Production Contract 由目标 `store-pnl-operating-analysis` Skill 所有；
- `references/test-sample-specification.md` 是本安装包的测试治理快照；
- 任何正式 Batch Evidence 必须同时记录 SUT Version 与 Test Harness Version；
- 更新版本时保留旧 Release Package，不覆盖历史包。

> **测试 Skill 的职责不是证明我们是对的，而是尽可能高效地找到我们哪里会错。**

# Test Sample Specification V0.2 — Installed Test Governance Snapshot

> 状态：`BUNDLED TEST GOVERNANCE SNAPSHOT`  
> 来源：Canonical `docs/testing/test-sample-specification.md`。  
> 用途：供 `store-pnl-batch-simulation` 在脱离 META 仓安装时执行最小测试治理。完整 Freeze Evidence 仍以主仓 Canonical 文档为准。

## 1. 总原则

测试采用四层：

```text
L1 Canonical / Boundary / Regression
L2 Scenario / Golden
L3 Property / Adversarial Random
L4 Multi-Agent Adversarial Review
```

> **小样本证明规则；大场景证明系统；随机样本寻找未知反例；确认后的反例永久转化为 Regression。**

> **Sample Count 不能替代 Coverage。**

## 2. 两种 Gate 不得混淆

### Contract / Methodology Baseline Freeze Gate

验证法律是否写清楚：数学、业务语义、状态路由、Population / Window / Materiality 等规则是否明确、自洽、可复现。

它不要求一个尚不存在的 Production Engine 先完整实现。

### Executable Implementation Acceptance Gate

验证真实 Engine / Runner / Skill 是否正确执行已经冻结的 Contract。

Contract Freeze 不自动等于 Implementation Acceptance。

## 3. 正式 Evidence 必须可复现

至少记录：

```text
candidate / sut version
input
expected result / expected property
verification method
actual output（若有 SUT）
tolerance
random seed（若适用）
run_id
```

聊天里“测过了”只能算 Exploratory Evidence。

## 4. State / Transition Coverage

当前 State：

```text
ABSENT
STANDARD
NET_ZERO_PRESENT
ZERO_GMV_NONZERO_GP
NEGATIVE_GMV
INVALID_OR_MISSING
OTHER_NONSTANDARD
```

理论矩阵 7×7。Contract Freeze 不要求 49 套大场景，但每个位置必须能够分类为：

```text
LEGAL_ROUTE
BOUNDARY_ROUTE
NOT_CONSTRUCTIBLE_UNDER_CURRENT_CONTRACT
N/A
```

每种不同合法路由 / Boundary 行为至少有一个确定性 Case。

必须覆盖：

```text
ABSENT → STANDARD = PURE_ENTRY
STANDARD → ABSENT = PURE_EXIT
NET_ZERO_PRESENT ≠ ABSENT
ABSENT → ABSENT 不制造虚假 Atom
OTHER_NONSTANDARD 不能由 AI 自行实例化
```

## 5. L1 Hard Rule Coverage

至少覆盖：

```text
State / Transition Routing
Presence / Zero Separation
GMV Additive Closure
Atomic Bennett
Parent Bennett
Parent Rate Bridge
Continuing Mix / Rate
Entry / Exit / Non-standard
Full Membership Replacement
Time Reversal
Derived Ratio Recalculation
Comparison Window Grain / Recalculation
Comparable Population Assembly
Individual Materiality
Registered Cohort Validation
Collective Materiality
Cross-View Non-Additivity
Boundary Stop
Language / Causal Boundary
```

Case 数只是参考，不是 Gate。

## 6. Integrated Contract Fixture

至少需要一个小型组合 Fixture，把：

```text
Scope
+ Window
+ Population / Comparable
+ Presence / State
+ Atomic / Parent Decomposition
+ Boundary
+ Materiality
+ Collective Materiality
+ Cross-View Guard
```

放在同一合法问题里，证明局部规则组合后不互相打架。

## 7. Scenario / Golden

Golden 适合完整实现和组合风险：

```text
Reality Profile
→ Scenario Truth
→ Synthetic Raw Data
→ Independent Oracle
→ Expected Result
→ Black-box SUT
```

Golden Truth 与 SUT 输入应权限分离。

至少区分：

```text
DESIGNED_GOLDEN
GENERATED_GOLDEN
ADVERSARIAL_GOLDEN
```

## 8. Property / Adversarial Random

随机测试必须分层构造，不做纯随机堆量。

每次 Run 保存：

```text
run_id
sut / implementation version
generator version
random_seed
sample_count
parameter_space
invariant_set
pass_count
fail_count
failed_case_ids
max_closure_error
```

失败样本保存完整可复现输入；通过样本可用 Seed + Generator Version 复现。

## 9. Coverage Matrix

正式维护：

```text
Hard Rule × Evidence Type
```

Evidence Type：

```text
Canonical
Boundary
Regression
Composition
Golden
Property
```

判断测试是否足够，优先看 Coverage，不看总 Case 数。

## 10. Finding 治理

```text
Finding
→ Concrete Failure Mechanism
→ Reviewer Self-Falsification
→ Cross Review
→ Human Adjudication
→ ACCEPT / DOWNGRADE / REJECT / DEFER
```

不得按模型数量投票。

只有 ACCEPT 的 Counterexample 才进入永久 Regression。

## 11. Contract Freeze 最低 Gate

最低 6 条：

1. Hard Rule Deterministic Coverage；
2. State / Transition Routing Closed；
3. 至少一个 Integrated Contract Fixture；
4. Accepted Counterexamples Closed；
5. Coverage Matrix 无未解释 High-risk Gap；
6. Freeze Evidence 可复现且 Version-bound。

任何适用 Hard Invariant Failure 阻止 Freeze。

## 12. Executable Implementation Acceptance 最低要求

至少：

1. L1 / Regression 对真实 SUT 全部 PASS；
2. Scenario / Golden 做 Black-box Expected vs Actual；
3. Property / Adversarial Random 达到约定规模并保留 Run Record；
4. Implementation-required 高风险 Coverage 有真实 SUT Evidence；
5. Invariant / Data Gate / Boundary / Language Guard 在 Runtime 中 Fail-closed。

## 13. 本安装包的执行纪律

- Batch Simulation Skill 只负责测试编排，不拥有 Production Contract；
- Production Expected Behavior 优先从目标 Skill 的 Contract 读取；
- 若 Expected Behavior 无法唯一确定，记录 `NON_DETERMINISTIC_CONTRACT`，不得测试者现场补法；
- 若 Oracle 不是独立实现，标记 `ORACLE_NOT_INDEPENDENT`；
- 真实数据只在内部授权环境使用；永久 Regression 必须匿名化 / 合成化。

> **测试的目的不是制造高通过率，而是用最小成本发现新的失败机制。**

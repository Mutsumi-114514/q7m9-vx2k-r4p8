# Repository Integration Audit — 2026-08-29

> 状态：`AUDIT EVIDENCE / NOT IMPLEMENTATION SOURCE`  
> 审计基线：`5c0b49c7178adbb75dc74418cad8997121481a97`  
> 审计范围：当前主仓全部已追踪文件；重点检查 Source of Truth、跨文档执行链、数据合同、Roll-up、Materiality、Collective Materiality、测试治理与历史文档隔离。  
> 目的：在继续 Golden Scenario 前，先排除“每条局部规则都合理，但叠加后执行系统版本漂移”的风险。

---

## 1. 审计结论

数学主干未发现需要推翻的新证据：

- Symmetric Bennett 保持；
- Parent `Total → Standard → Continuing` 保持；
- Entry / Exit / Non-standard Bridge 保持；
- Time Reversal Mapping 保持；
- Derived Ratio Never Rolls Up 保持；
- Gross Movement / Offset 保持。

但发现 **4 个需要在副作用测试前先修复的仓库级问题**，其中 2 个属于高风险治理问题。

---

## 2. AUDIT-F001 — Source of Truth 分裂

Severity: `HIGH`

### 现象

仓库中同时存在：

```text
production-shadow-decomposition.md
materiality-gate.md
collective-materiality.md
```

后两者被定义为 `CURRENT PRODUCTION ADDENDUM`，且 Materiality 文件显式声明在冲突时覆盖旧 Production Baseline。

与此同时：

- README 的 Source of Truth 列表没有 Materiality / Collective Materiality；
- project-handoff 仍把旧 `production-shadow-decomposition.md` 描述为唯一当前有效算法基线；
- Execution Architecture 的运行链没有 Materiality / Collective Materiality。

### 风险

新的 AI / 实现者可能：

1. 完整阅读 README 与 Handoff；
2. 严格执行旧 Production Baseline；
3. 仍然合法地漏掉 GS-001 / GS-002 已裁决规则。

这属于：

> **Document-local correctness, system-level version drift。**

### 修复

新增：

```text
docs/methodology/production-system-contract.md
```

作为唯一当前运行入口，统一数学 Kernel + Materiality + Collective Materiality + Cross-View Guard。

---

## 3. AUDIT-F002 — 决策层规则未进入执行合同

Severity: `HIGH`

### 现象

现有 `skill-execution-architecture.md` 与 `impact-ledger-and-rollup.md` 的 Parent-dependent / Contextual 指标清单仍主要包含：

```text
Parent Bennett
Mix / Rate
Entry / Exit
Bridge
Contribution
Offset
```

没有把：

```text
Individual Materiality
Collective Materiality
Structured Long-tail
```

作为 Contextual Engine 的正式产物和运行阶段。

### 风险

项目原则是：

> `Code enforces / Schema constrains / Prompt explains`

但两轮 Golden 新增规则若只停留在 Addendum，自然语言 Prompt 很可能成为唯一约束，违背既有工程原则。

### 修复

统一 System Contract 明确：

```text
Persistent Atomic Ledger
≠
Contextual Materiality / Attention
```

并把 Materiality、Collective Materiality、Long-tail、Attention 纳入正式执行链和最小结果字段。

---

## 4. AUDIT-F003 — Cross-View 解释层存在认知重复计数风险

Severity: `MEDIUM`

### 现象

既有 Roll-up 合同已经明确：

> 不同投影可以分别完整闭合，但不能再次相加。

Materiality / Collective Materiality 新增后，同一底层变化可能同时出现在：

```text
门店
渠道
店型
分部
规模带
Derived View
```

原仓库没有把“投影不可相加”明确延伸到 Finding / Attention 层。

### 风险

可能出现：

```text
一个真实结构变化
→ 五个 View 同时命中
→ 被 AI 汇报成五个独立问题
```

数学没有 Double Count，但业务阅读产生 Cognitive Double Count。

### 修复

统一 System Contract 新增：

```text
Cross-View Non-Additivity Guard
```

在 Finding Consolidation 尚未测试通过前：

- 跨 View 不得数值相加；
- 不得自动声明独立原因；
- 无法判定关系时使用 `OVERLAP_UNKNOWN`。

注意：真正的 Finding Consolidation 算法仍未冻结，必须进入副作用测试。

---

## 5. AUDIT-F004 — Reality Profile 存在两个已知算术冲突但未显式标记

Severity: `MEDIUM`

### 冲突 A：店型数量

Reality Profile：

```text
store_count = 218
```

但店型数量：

```text
74 + 63 + 44 + 15 + 14 + 7 + 3 = 220
```

因此这些数量不能同时作为一个 218 店 Universe 的精确互斥分布。

### 冲突 B：渠道覆盖

Reality Profile 同时记录：

```text
186 / 218 家门店四渠道齐全
星选覆盖率约 75%
```

如果 186 家四渠道齐全，则星选覆盖率至少为：

```text
186 / 218 = 85.3%
```

因此两条不能在同一统计 Universe / Period 下同时为精确事实。

### 风险

Golden Generator 若把全部画像参数机械当硬约束，会生成不存在的数学目标，或者为了同时满足目标而偷偷扭曲其他分布。

### 修复

新增 Reality Profile Validation Note：

- 保留原始画像，不篡改来源；
- 精确计数与近似口径冲突时不得自动同时强制；
- Generator 必须记录采用哪条约束、舍弃哪条以及原因。

---

## 6. 已检查但未发现新增确定性缺陷的区域

### Data Contract

- `ABSENT ≠ NET_ZERO_PRESENT` 正确；
- `STANDARD` / Zero-GMV / Negative GMV 路由边界一致；
- Rate Never Rolls Up 一致；
- 官方控制数优先原则一致。

### Production Math

- Bennett / Mix-Rate Closure 定义一致；
- Entry↔Exit Time Reversal Mapping 一致；
- Full Membership Replacement 边界一致；
- Parent Bennett zero denominator gate 一致。

### Historical Review

Round 2 / Round 3 顶部均存在：

```text
HISTORICAL REVIEW / NOT IMPLEMENTATION SOURCE
```

历史公式污染风险已被显式隔离。

### Multi-Agent Review Governance

Blind / Staging / Human Adjudication 逻辑自洽；当前需要在未来正式 Review Scope 中补入 Decision Precision / Materiality / Collective Materiality，但不影响现有隔离协议本身。

---

## 7. 审计后尚未解决、必须由副作用测试回答的问题

这些不是本轮直接修复的确定性文档 Bug，而是系统交互风险：

1. Materiality 降权 + Collective Materiality 升权会不会形成反复升降级；
2. 多个 Registered View 会不会造成 Finding Explosion；
3. 同一底层现象在门店 / 渠道 / 店型 / 分部投影中会不会被重复汇报；
4. Structured Long-tail 是否会因为拆得过细而重新制造 Attention Explosion；
5. 不同 Cohort 高度重叠时，如何判断支持关系、重叠关系或独立关系；
6. Offset、Materiality、Collective Materiality 同时存在时，最终排序是否稳定；
7. Cross-View Guard 是否足以在没有完整 Consolidation 算法时阻止错误业务表达。

这些问题进入：

> **System Interaction / Side-effect Golden Test**。

---

## 8. 审计裁决

```text
Core Math Rewrite = NO
Data Contract Rewrite = NO
Source-of-Truth Consolidation = YES
Decision-layer Execution Contract = YES
Cross-View Safety Guard = YES
Reality Profile Validation = YES
System Side-effect Test = REQUIRED
```

本轮原则：

> **不因为担心复杂性而撤回已通过反例验证的规则；先消除规则之间的版本与边界歧义，再用系统级 Golden 测试判断组合副作用。**

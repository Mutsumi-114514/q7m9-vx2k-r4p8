# Multi-Agent Review Protocol：多方 AI 独立评审与交叉验证协议

> 定位：经营分析 Skill / 算法 / 执行语义在进入 Production 前的评审治理协议。  
> 目标：降低单一模型、单一对话、单一思路产生的共同盲区，并保证独立评审、测试生成、交叉复核和最终裁决可追踪。  
> 核心原则：**主仓是法典与档案馆；临时仓是一次性独立考场。**

---

## 1. 为什么需要多方评审

单一 Reviewer 即使能力很强，也可能因为：

- 已有上下文形成 Anchoring（锚定）；
- 长对话产生 Attention Drift（注意力漂移）；
- 与项目长期共同演化而形成 Shared Assumption Blind Spot（共同假设盲区）；
- 测试样本覆盖不足；
- 数学正确但业务语义错误；
- 文档正确但执行路径错误；

而稳定遗漏同一类问题。

因此正式评审不把“多个 AI 看过”视为充分条件，而要求：

> **不同 Reviewer 在隔离上下文中独立评审、独立生成测试，再进行交叉复核，最终由人裁决。**

---

## 2. 两类仓库与一个暂存区

### 2.1 主仓：永久 Source of Truth

主仓永久保存：

- 当前 Production Baseline；
- 数据合同；
- 执行架构；
- 已完成独立阶段后的 Reviewer 原始意见与测试样本；
- Cross Review（交叉评审）结果；
- Human Adjudication（人工裁决）结果。

主仓不是盲审考场。

### 2.2 临时仓：Disposable Review Environment

临时仓只服务于 **一个 Reviewer 的一次独立评审**。

规则：

> **One Reviewer × One Independent Review = One Disposable Repository.**

临时仓只包含本轮允许 Reviewer 看到的材料。

Reviewer 完成评审后：

1. 导出评审意见与测试样本；
2. 保存到独立阶段的 Staging（暂存区）；
3. 删除整个临时仓；
4. 下一位 Reviewer 使用重新创建的全新临时仓。

> **Review Repository Is Disposable Infrastructure, Not Knowledge Storage.**

### 2.3 Staging：独立阶段的隔离暂存区

在所有 Blind Reviewer 独立交卷之前，Reviewer 结果默认先保存到：

> **后续 Reviewer 无法访问的 Staging。**

Staging 可以是：

- 本地文件夹；
- 不向 Reviewer 开放的私有目录 / 仓库；
- 其他能够保证后续 Reviewer 无权读取的存储位置。

推荐流程不是“Reviewer A 一交卷就立刻写回主仓”，而是：

```text
A 交卷 → Staging
B 交卷 → Staging
C 交卷 → Staging
全部独立交卷完成
↓
一次性归档主仓
↓
Cross Review
```

这样可以避免后续 Reviewer 即使意外拥有主仓读取能力，也提前看到前序答案。

---

## 3. 为什么不能重复使用同一个临时仓

即使删除上一位 Reviewer 的文件，Git commit history 仍可能保留历史内容。

因此禁止：

```text
Test Repo
→ Reviewer A
→ 删除 A 文件
→ 同一个 Repo 给 Reviewer B
```

正式做法：

```text
Temp Repo 1 → Reviewer A → 导出结果 → 删除 Repo 1
Temp Repo 2 → Reviewer B → 导出结果 → 删除 Repo 2
Temp Repo 3 → Reviewer C → 导出结果 → 删除 Repo 3
```

通过仓库生命周期而不是 Prompt 保证主要隔离边界。

> **Permission / Environment enforces; Prompt explains.**

---

## 4. 同一轮所有 Reviewer 必须评同一个冻结版本

每轮开始前先冻结主仓 Baseline Commit：

```text
baseline_commit = <SHA>
```

这一轮所有 Blind Reviewer 必须针对同一个 SHA。

禁止：

```text
Reviewer A 评 v1
→ 发现问题
→ 立即修改 Production
→ Reviewer B 评 v2
```

否则不同 Reviewer 的发现无法直接比较。

规则：

> **本轮独立评审全部提交、Cross Review 完成、人工裁决结束前，被评 Baseline 冻结。**

---

## 5. Blind Review 的上下文隔离

Blind Reviewer 不应读取：

- 历史 Round 1 / Round 2 / Round 3 评审；
- 已知缺陷列表；
- 其他 Reviewer 的意见；
- 既往裁决结论；
- 当前项目长对话历史；
- 主仓中与本轮盲审无关的历史解释。

Blind Review 应尽量使用：

```text
新对话
+
全新临时仓
+
指定冻结 Baseline
+
最小必要评审材料
+
无主仓 / 其他 Reviewer 结果读取能力
```

因此：

> **Source of Truth ≠ Context of Truth。**

---

## 6. HARD_BLIND 与 SOFT_BLIND 必须区分

### `HARD_BLIND`：硬隔离盲审

满足：

```text
Reviewer 技术上无权读取主仓
Reviewer 技术上无权读取其他 Reviewer 结果
Reviewer 不处于继承历史上下文的旧对话 / Project 环境
Reviewer 只能读取本次临时仓允许材料
```

这是正式三方评审的首选模式。

### `SOFT_BLIND`：软隔离盲审

Reviewer 实际拥有更多读取能力，只靠 Prompt 要求“不读取”。

例如：

```text
已连接整个 GitHub 账号
但提示词要求只读临时仓
```

这只能算软约束。

规则：

> 正式 Round 应尽量使用 `HARD_BLIND`；若只能使用 `SOFT_BLIND`，必须在 `baseline-ref.md` 中显式记录，不得把它冒充完全独立盲审。

---

## 7. “新对话”不自动等于“新上下文”

如果新对话仍处于：

- 同一个长期 Project；
- 共享项目上下文；
- 可恢复历史 Memory；
- 已连接可访问 Review History 的数据源；

则可能仍不是 Blind Reviewer。

正式 Blind Reviewer 应尽量满足：

```text
fresh_conversation = true
historical_project_context = unavailable
main_repo_access = false
other_review_results_access = false
```

若无法完全满足，必须降级记录为 `SOFT_BLIND`。

---

## 8. Reviewer 角色：共同必答题 + 专项攻击题

多个 Reviewer 不应只收到完全相同的“帮我看看有没有问题”，但也不能完全拆成互不重叠的职责。

### 8.1 所有 Reviewer 的共同必答题

至少都检查：

```text
Closure
Time Reversal
Entry / Exit
Absent / Zero / Null
Negative / Zero denominator
Atomic Attribution vs Parent Re-decomposition
Roll-up legality
Semantic Validity
```

这样同一个重大错误有机会被多个独立模型重复捕获。

### 8.2 专项角色

#### Mathematical Reviewer

强化：

- Closure；
- Symmetry；
- Time Reversal；
- 顺序依赖；
- 权重与分母；
- 极端数学状态。

#### Business Semantic Reviewer

强化：

- 数学闭合但业务表达荒谬；
- Parent / View / Scope 语义漂移；
- Attribution 被误写成 Causation；
- Entry / Exit / Mix / Rate 的业务解释。

#### Test / Engineering Reviewer

强化：

- 状态机组合；
- Boundary；
- Null / Zero / Negative；
- Roll-up / Re-decomposition；
- Schema / Type；
- 执行顺序；
- 随机与对抗测试样本。

模型名称不是协议核心，**角色差异与上下文隔离才是核心**。

---

## 9. 每个 Reviewer 必须独立生成测试样本

正式评审不能只输出文字意见。

每个 Reviewer 至少同时交付：

```text
findings.md
测试样本 / 测试表
预期结果
失败条件
```

随机或批量测试还应记录：

```text
random_seed
generator_version
case_id
```

以便复现。

测试应优先覆盖：

- Pure Scale；
- Pure Rate；
- Pure Mix；
- Strong Offset；
- Entry；
- Exit；
- `ABSENT` vs `NET_ZERO_PRESENT`；
- Zero GMV + Non-zero GP；
- Entry + 非标准状态；
- Exit + 非标准状态；
- Full Membership Replacement；
- Negative Margin；
- Negative GMV；
- Tiny Denominator；
- Parent GMV = 0；
- Time Reversal，含 Entry↔Exit 映射；
- Atomic Attribution → Parent View；
- YTD / 时间聚合；
- 其他 Reviewer 自主发现的边界状态。

鼓励 Reviewer 自主生成大量随机或对抗样本，而不是只验证仓库已经列出的示例。

---

## 10. 主仓评审归档结构

评审原始意见可以错误，因此必须和 Production Baseline 分层保存。

建议：

```text
reviews/
└─ round-XX/
   ├─ baseline-ref.md
   ├─ reviewer-a/
   │  ├─ findings.md
   │  └─ tests.md
   ├─ reviewer-b/
   │  ├─ findings.md
   │  └─ tests.md
   ├─ reviewer-c/
   │  ├─ findings.md
   │  └─ tests.md
   ├─ cross-review/
   └─ adjudication.md
```

`baseline-ref.md` 至少记录：

```text
review_round
baseline_commit
review_date
review_scope
reviewers
blind_mode = HARD_BLIND | SOFT_BLIND
reviewer_access_boundary
```

Reviewer 文件属于 **Review Evidence（评审证据）**，不是当前算法事实。

---

## 11. Cross Review：独立交卷后才互相看答案

所有 Blind Reviewer 完成独立提交后：

1. 将 Staging 中的原始结果归档主仓；
2. 再进入 Cross Review。

此阶段允许各 Reviewer 阅读其他人的：

- Findings；
- 测试样本；
- 预期结果；
- 质疑点。

Cross Review 重点回答：

1. 其他 Reviewer 的 Finding 是否真实；
2. 测试样本是否有效；
3. 是否存在相同问题的不同表述；
4. 是否存在互相冲突的结论；
5. 某个 Finding 是否可以推出更深层缺陷；
6. 是否出现新的组合反例。

在此阶段，信息共享不再属于污染，而是评审目标的一部分。

---

## 12. Human Adjudication：AI 发现不等于项目结论

最终由人裁决每个 Finding。

至少使用：

```text
ACCEPT
REJECT
NEEDS_MORE_TESTING
DEFER
```

示意：

| Finding | 来源 | 裁决 | 说明 |
|---|---|---|---|
| F01 | Reviewer A | ACCEPT | 真实 Closure Failure |
| F02 | Reviewer B | REJECT | 测试前提违反数据合同 |
| F03 | Reviewer C | NEEDS_MORE_TESTING | 需要随机样本验证 |

只有 ACCEPT 的 Finding 才允许修改 Current Production Baseline。

> **Only adjudicated findings enter the Source of Truth.**

---

## 13. 一轮标准操作流程

```text
① Freeze Baseline Commit
        ↓
② 为 Reviewer A 创建全新临时仓
        ↓
③ A 独立评审 + 独立造测试
        ↓
④ A 结果导出到隔离 Staging
        ↓
⑤ 删除 A 临时仓
        ↓
⑥ 为 Reviewer B 创建全新临时仓
        ↓
⑦ B 结果导出到隔离 Staging
        ↓
⑧ Reviewer C 同理
        ↓
⑨ 全部 Reviewer 独立交卷
        ↓
⑩ Staging 结果一次性归档主仓
        ↓
⑪ Cross Review
        ↓
⑫ Deterministic Runner 验证可执行样本
        ↓
⑬ Human Adjudication
        ↓
⑭ ACCEPT Finding 修复 Production
        ↓
⑮ 新 Baseline 进入下一轮
```

---

## 14. 当前阶段保持简单

V0.1 不要求提前开发：

- 自动 Review Factory；
- YAML 评审编排器；
- 自动建删仓服务；
- MCP 权限代理；
- 多 Agent 自动通信平台。

当前先用人工流程跑通：

> **冻结版本 → 建临时仓 → 发链接 → AI 评审 → 导出到隔离 Staging → 删除临时仓 → 下一位 Reviewer → 全部交卷后统一归档。**

只有当人工操作本身成为稳定瓶颈时，再自动化。

原则：

> **先验证治理机制本身有价值，再工程化治理机制。**

---

## 15. 最终原则

多方评审不是“找更多 AI 投票”。

真正要降低的是 Correlated Failure（相关性失败）。

因此需要同时制造：

- 模型差异；
- Reviewer 角色差异；
- 对话上下文差异；
- 测试样本差异；
- 独立评审与交叉评审的阶段差异。

核心纪律：

> **Reviewer Independence Is a Context, Permission and Environment Design Problem.**

以及：

> **主仓保存知识；临时仓提供隔离；Staging 防止提前泄题；独立评审先于交叉评审；人工裁决先于 Production 修改。**

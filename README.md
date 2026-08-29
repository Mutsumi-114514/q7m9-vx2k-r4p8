# Operating Analysis Skill Distribution Mirror

这是 `operating-analysis-skills` 的 **Skill-only Distribution Mirror**。

用途：让内部 AI / 外部审查者只读取与当前 Skill 安装、运行和测试有关的内容，避免整个 META 仓带来的 Context Dilution。

当前同步源：

```text
Mutsumi-114514/operating-analysis-skills
main @ c79c2f849d14c85384981e21bb3757e8923260dd
```

## 当前结构

```text
skills/
├─ runtime/
│  └─ store-pnl-operating-analysis/
└─ testing/
   └─ store-pnl-batch-simulation/

skill-packages/
├─ runtime/
│  └─ store-pnl-operating-analysis-v0.1.0-field-trial/
└─ testing/
   └─ store-pnl-batch-simulation-v0.1.0-field-trial/

docs/changelog/
└─ skill-release-log.md
```

本镜像**不包含**主仓的 Roadmap、Retrospective、Research、历史 Audit、Review History 与 Legacy Prototype。

安装时请使用 `skill-packages/` 中带版本号的快照，或 `skills/` 中的 Current 工作副本。

> **Current 用于工作，Versioned Package 用于交付；META 研究不进入本镜像。**

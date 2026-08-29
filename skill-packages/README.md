# Skill Packages

这里保存**不可覆盖的版本化交付快照（Immutable Versioned Packages）**。

与 `skills/` 的区别：

```text
skills/
= Current 工作面

skill-packages/
= Versioned 交付面
```

## 命名规则

```text
<skill-name>-v<version>/
```

例如：

```text
runtime/store-pnl-operating-analysis-v0.1.0-field-trial/
testing/store-pnl-batch-simulation-v0.1.0-field-trial/
```

目录内部仍保持标准 Agent Skill 文件名：

```text
SKILL.md
INSTALL.md
references/
```

不要将入口改名为 `SKILL-v0.1.md`，否则会破坏通用 Skill 发现规则。

如果需要 ZIP 分发，ZIP 文件名使用版本化名称：

```text
store-pnl-operating-analysis-v0.1.0-field-trial.zip
store-pnl-batch-simulation-v0.1.0-field-trial.zip
```

> **包名区分版本，包内结构遵守标准。**

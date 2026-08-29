# Skills 目录规则（Distribution Mirror）

本镜像只保留当前可安装 Skill。

```text
skills/
├─ runtime/      # 当前正式运行 Skill
└─ testing/      # 当前测试 / 压测 Skill
```

主仓中的 Legacy Prototype 不进入本分发镜像，避免 Runtime 混淆。

标准 Skill 目录内部保持：

```text
SKILL.md
INSTALL.md
references/
```

版本号写入 Skill metadata；发布包目录和建议 ZIP 文件名带版本号。

> **Current 是工作面；skill-packages 是交付面。**

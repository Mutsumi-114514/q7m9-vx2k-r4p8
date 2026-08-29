# 安装说明 — store-pnl-batch-simulation

这是 `store-pnl-operating-analysis` 的标准可安装测试 Skill。

## 目录结构

```text
store-pnl-batch-simulation/
├─ SKILL.md
├─ INSTALL.md
└─ references/
   └─ test-sample-specification.md
```

## 依赖

运行前必须同时安装：

```text
store-pnl-operating-analysis
```

本测试 Skill 不复制 Production Contract。被测 Skill 自己拥有其 Production / Data / Scope Contract；测试 Skill 只拥有测试编排与 Testing Governance。

## 安装

保持整个 `store-pnl-batch-simulation/` 目录结构不变。

对于支持 ZIP 上传的 Agent Skills 平台：

1. 将整个目录压缩；
2. 建议下载 / 分发文件名使用：

```text
store-pnl-batch-simulation-v0.1.0-field-trial.zip
```

3. ZIP 解压后顶层目录应包含 `SKILL.md`；
4. 同时确认 `store-pnl-operating-analysis` 已安装且可被调用。

对于文件系统型 Skill 运行时，将整个目录放入对应 Skills Root。

## 冒烟测试

```text
使用 store-pnl-batch-simulation，对已安装的 store-pnl-operating-analysis 做一轮小规模冒烟测试：先扫描数据版图，再生成 20 个分层真实 Query；遇到数字水印、Population 或 Hard Invariant 错误立即停止并给出最小反例。
```

## 正式批跑建议

第一轮可使用：

```text
100–300 个 Real-data Shadow Query
+
500–2,000 个 Reality-shaped Simulation Case
```

这是 Field Trial 起步规模，不是通过标准。Coverage 和 Failure Mechanism 优先于 Sample Count。

## 安全边界

本 Skill 标记为 `INTERNAL DATA ONLY`：

- 不把真实经营数据复制到外部模型；
- 不把真实门店与金额写入公开仓库；
- Regression 必须匿名化或合成化；
- 原始 Production 数据只读。

## 版本与发布

标准 Skill 入口文件必须保持名称 `SKILL.md`，因此**不要把入口改名为带版本号的文件**。

版本号放在：

- `SKILL.md` metadata；
- Release Package / ZIP 文件名；
- `skill-packages/` 中的版本化目录；
- Skill Release Log。

这样同时满足 Agent Skill 标准和历史版本可区分要求。

# Skill Release Log

## 2026-08-29

### store-pnl-operating-analysis — `v0.1.0-field-trial`

- 标准可安装 Runtime Skill；
- `SKILL.md + INSTALL.md + references/`；
- 内置 Production / Data / Scope-Population Runtime Snapshot；
- 包含 Analysis Fingerprint、Structured Result、Boundary / Cross-View / Evidence Guard。

版本化交付目录：

```text
skill-packages/runtime/store-pnl-operating-analysis-v0.1.0-field-trial/
```

建议 ZIP 文件名：

```text
store-pnl-operating-analysis-v0.1.0-field-trial.zip
```

### store-pnl-batch-simulation — `v0.1.0-field-trial`

- 标准可安装 Test Harness Skill；
- SUT 固定为 `store-pnl-operating-analysis`；
- 包含 Real-data Shadow Batch、Reality-shaped Simulation、Metamorphic Test、Independent Oracle、Counterexample Minimization；
- 内置精简 Testing Governance Snapshot；
- 明确 `ORACLE_NOT_INDEPENDENT`、Hard Invariant、Failure Classification。

版本化交付目录：

```text
skill-packages/testing/store-pnl-batch-simulation-v0.1.0-field-trial/
```

建议 ZIP 文件名：

```text
store-pnl-batch-simulation-v0.1.0-field-trial.zip
```

## 版本纪律

- 标准入口必须保持 `SKILL.md`；
- 版本写入 metadata；
- Package / ZIP 名称带版本号；
- 新版本新增 Package，不覆盖旧 Package；
- 本镜像不保存 Legacy Prototype 和 META 研究材料。

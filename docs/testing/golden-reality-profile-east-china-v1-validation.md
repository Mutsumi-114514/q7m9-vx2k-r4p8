# Golden Reality Profile V1 — Validation / Normalization Note

> 状态：`CURRENT TEST INPUT VALIDATION NOTE`  
> 对象：`docs/testing/golden-reality-profile-east-china-v1.md`  
> 来源：Repository Integration Audit，2026-08-29。  
> 原则：保留原始匿名画像，不篡改来源；发现内部算术冲突时，Generator 不得把冲突项同时当作硬约束。

---

## 1. 店型计数不闭合

Reality Profile 同时记录：

```text
store_count = 218
```

以及：

```text
中小店 = 74
城旗 = 63
核心店 = 44
自营门店 = 15
已关店 = 14
超体 = 7
非门店 = 3
```

上述店型数量合计：

```text
220
```

因此当前不能证明这些数字来自同一个时点、同一个互斥口径的 218 店 Universe。

### Generator 规则

在未重新取得统一口径前：

- `store_count = 218` 可作为组织规模硬约束；
- 店型数量只能作为 Distribution Shape / Soft Reference；
- 若 Scenario 需要精确 218 店互斥店型分配，必须记录 normalization rule；
- 不得静默修改原始 Reality Profile 数字并宣称为真实口径。

GS-001 曾临时采用：

```text
中小店 74 → 73
城旗   63 → 62
```

仅用于合成数据闭合，不代表真实数据裁决。

---

## 2. 渠道覆盖口径冲突

Reality Profile 同时记录：

```text
186 / 218 家门店四渠道齐全
```

以及：

```text
星选覆盖率约 75%
```

若 186 家门店确实四渠道齐全，则任何一个原子渠道的门店覆盖率都至少为：

```text
186 / 218 ≈ 85.3%
```

因此“186 家四渠道齐全”与“星选覆盖约 75%”不能在同一时点 / 同一门店 Universe 下同时作为精确硬约束。

### Generator 规则

在未重新取得统一口径前：

- 精确计数优先作为 Scenario 可复现约束；
- “约 75%”只作为 Soft Reference / Alternate Scope Signal；
- 若 Generator 采用 186 家四渠道齐全，则不得再强制星选覆盖 75%；
- 若未来证实 75% 属于另一 Period / Scope，应显式记录该 Context。

---

## 3. 不影响 Production Data Contract

上述冲突只影响 Reality Profile 如何驱动 Synthetic Generator。

它们不修改：

```text
Atomic Grain = 月份 × 门店 × 渠道
Atomic Channels = 地采 / 集采 / 万家 / 星选
大集采 = 集采 + 万家
Period State Contract
```

也不影响真实生产数据的官方 Control Total。

---

## 4. 正式 Golden Package 要求

任何使用该 Reality Profile 的 Golden Scenario，Manifest 至少增加：

```text
reality_profile_version
reality_profile_validation_note
normalization_rules
hard_constraints_used
soft_references_used
known_profile_conflicts
```

核心原则：

> **现实画像可以不完美，但合成测试必须对自己采用的假设完全透明。**

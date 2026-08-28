# Golden Reality Profile V1：华东一区匿名现实结构画像

> 状态：`INTERNAL_GOLDEN_SIMULATION_INPUT`  
> 用途：为单店损益 MVP 的 Golden Scenario Generator 提供现实形状、业务语义与对抗测试约束。  
> 数据来源：6 个内部文件、14,293 行真实数据的匿名统计结果 + 业务人工确认。  
> 安全边界：本文不保存真实门店名称、真实 GMV / 毛利金额、真实业务明细行；仅保存结构数量、比例、分位数、倍数、布尔事实与业务语义。  
> 重要说明：本文件是 Reality Profile（现实画像），不是 Golden Expected Result（黄金答案）。真实结构参数只能约束合成数据“像真实世界”，不能作为算法正确性的证明。

---

## 1. 当前已确认的事实颗粒度

### 1.1 标准门店事实 Grain

当前单店损益标准 Atomic Grain（最细分析原子）冻结为：

```text
月份 × 门店 × 渠道
```

`门店类型` 不进入 Atomic Grain，而是门店属性（Attribute）。

人工确认：同一个 `月份 × 门店 × 渠道` 不会同时出现多个不同门店类型。

### 1.2 门店类型属性

- 门店类型基本固定，但长期并非绝对不可变化。
- 门店类型与门店规模存在明显相关性：超体 / 城旗 / 中小店等店型具有不同的大致规模倾向。
- 但同一店型内部规模离散仍然很大，因此不得把店型机械映射成固定规模。
- Golden Generator 应采用：

```text
店型 → 店型规模分布 → 门店个体规模
```

而不是：

```text
店型 → 唯一固定规模
```

### 1.3 Raw Row

当前画像结果显示标准门店体系下：

```text
allow_multiple_raw_rows_per_key = NO
raw_rows_per_atom_typical_range = 1
```

但测试规范本身仍允许通过人工构造多 Raw Row 原子来验证：正负冲销、Presence 与净额分离、不自动去重等规则。Reality Profile 不限制测试系统构造更强的对抗样本。

---

## 2. 组织与规模结构

```text
division_count = 11
store_count = 218
channel_count = 4
store_type_count = 8
base_month_count = 12   # 2025-01 ~ 2025-12
current_month_count = 7 # 2026-01 ~ 2026-07
```

### 2.1 各分部门店数量

```text
P25 = 17
P50 = 21
P75 = 22
range = 10–29
```

### 2.2 店型数量

匿名画像中的当前数量：

```text
中小店 = 74
城旗 = 63
核心店 = 44
自营门店 = 15
已关店 = 14
超体 = 7
非门店 = 3
慧采 = 0 个标准门店ID实体（存在分部级特殊行）
```

### 2.3 门店规模分布

以中位数门店 = 1.00x 标准化：

```text
P10 = 0.00x
P25 = 0.32x
P50 = 1.00x
P75 = 2.34x
P90 = 3.80x
Max = 21.5x
```

集中度：

```text
Top1 GMV share = 5.5%
Top3 GMV share = 13.2%
Top5 GMV share = 18.8%
Top10 GMV share ≈ 28%
```

解释边界：

- `Store Weight Concentration（少数门店垄断总体权重）` 不算特别高；
- `Store Size Skewness（门店规模分布偏斜）` 很高；
- 这两个概念不能合并成一个 `store_concentration`。

建议 Generator 分别保存：

```text
store_weight_concentration = LOW_TO_MEDIUM
store_size_skewness = HIGH
```

### 2.4 分部规模分布

以中位数分部 = 1.00x：

```text
P25 = 0.85x
P50 = 1.00x
P75 = 1.22x
Max = 1.82x
Top1 division share = 15.2%
Top3 division share = 40.8%
```

现实形状：分部之间相对均衡，明显弱于门店层面的长尾分化。

---

## 3. 渠道结构

原子渠道：

```text
地采
集采
万家
星选
```

总体 GMV 结构：

```text
地采 = 71.0%
集采 = 15.4%
万家 = 11.9%
星选 = 1.7%
```

其他结构：

```text
channel_mix_heterogeneity = HIGH
highly_single_channel_store_ratio = 16%  # 最大渠道占比 > 90%
inactive_store_channel_combination_ratio = 8.4%
channel_entry_exit_exists = NO（历史画像整体口径；不代表原子层永不新增渠道）
```

现实覆盖：

- 186 / 218 家门店四渠道齐全；
- 地采几乎每店都有，覆盖率 95%+；
- 星选覆盖率最低，约 75%。

### 3.1 渠道结构的动态语义

人工确认：

- 同一家门店的渠道结构短期通常相对稳定；
- 长期可以发生很大的结构变化；
- 大变化可能来自集团政策变化；
- 因此渠道结构不得被冻结成全年固定属性，也不得每月完全独立随机。

Golden Generator 宜采用：

```text
门店基础渠道结构
    → 月度小幅波动
    → 少数阶段出现结构迁移 / Policy Regime Change
    → 在新水平附近重新稳定
```

这里的“政策导致”只作为现实可能性，不允许算法仅凭数据自动下因果结论。

### 3.2 店型与渠道结构

人工确认：不同店型的渠道结构可能存在明显差异，但不是硬规则。

因此 Generator 可以让店型影响渠道占比的先验分布，但必须保留大量同店型内部差异和反例。

### 3.3 大集采

业务派生分组：

```text
大集采 = 集采 + 万家
```

`大集采` 是 Derived Channel Group（派生渠道组），不是第五个 Atomic Channel。

测试必须防止把 `集采 + 万家 + 大集采` 同时汇总造成重复计算。

---

## 4. 时间结构

```text
seasonality_strength = HIGH
```

相对低月 = 1.00x 的现实画像：

```text
高月：1月 ≈ 1.92x
      6月 ≈ 2.24x–2.55x
      11月 ≈ 2.06x

低月：2月 ≈ 1.00x–1.17x
      4月 ≈ 1.19x–1.32x

typical_monthly_volatility_range ≈ 1.0x–2.55x
```

其他时间事实：

```text
store_entry_exists = YES
store_exit_exists = YES
ytd_vs_single_month_direction_reversal_exists = YES
```

现实画像中约 35% 门店存在 H1 同比方向与 6 月单月同比方向相反。

开关店通常零星发生，约 1–5 家 / 月，并非单月集中大规模开关。

---

## 5. 门店生命周期与销售生命周期必须分离

人工确认：

- 只要是门店，门店整体会有销售；
- 某个 `门店 × 渠道` 可以从无销售变为有销售；
- 因此 Atomic Grain 上 `ABSENT → STANDARD` 通常只表示“该门店新增这个渠道的销售”，不能自动翻译为“新店开业”；
- 门店正式开业前可能已经通过线上渠道产生销售；
- 门店关店后仍可能产生退货、冲销等销售记录；
- 旧店关闭、新店开启之间还可能存在一段两者同时经营的 Overlap（存续重叠期）；
- 数据中存在可靠的开业时间、关店状态字段，但当前单店损益分析并不以门店生命周期为重点。

因此禁止：

```text
First Sale Record = Store Opening
Last Sale Record  = Store Closure
```

只有结合明确的生命周期字段，才允许将原子 Entry / Exit 进一步解释为开店 / 关店。

Golden Scenario 应至少包含：

1. 老店新增渠道：Atomic Entry，但不是 Store Entry；
2. 新店正式开业前已有线上销售；
3. 关店后仍出现退货 / 冲销；
4. 旧店与新店存在一段并行存续期。

---

## 6. ABSENT 与数据完整性

人工确认：当前数据规范性较高。

在标准门店 Atomic Grain 下：

```text
该 月份 × 门店 × 渠道 没有记录
≈ 该原子当月确实没有业务
```

因此在当前数据域与通过控制数校验的前提下，可以将：

```text
ABSENT（无原子记录）
```

作为可执行业务状态，而不是默认怀疑数据漏提。

注意：这是当前数据源的 Reality Assumption，不应被泛化为所有未来数据源的永久真理。新数据源仍应经过 Data Gate / Coverage Check。

---

## 7. 特殊行：无门店 ID 但必须保留

现实数据另有约 1,606 行缺少 `超星门店id` 的特殊记录，主要来自：

- 慧采；
- 非门店；
- 部分自营门店；
- 已关店残留等。

其大部分金额很小，不可能存在“没有门店同时又重大”的常规业务记录。

但人工确认：底表最终一定要与考核报表总数闭合，因此这类记录不能为了让 Schema 更漂亮而直接删除。

建议语义：

```text
Special Rows / Organization-level Facts
```

MVP 中：

- 不作为主要 WHERE / WHY 分析对象；
- 不硬塞进某家标准门店；
- 不默默丢弃；
- Parent Control Total 必须包含；
- 可单独进入 Special / Boundary 路径。

Golden Scenario 应少量保留这类记录，以测试：

1. 是否破坏 Parent 总数闭合；
2. 是否被错误归属到标准门店；
3. 是否因特殊形态被误判成重大经营问题。

---

## 8. 毛利率现实分布

现实门店月画像：

```text
margin_rate_P10 = 8.2%
margin_rate_P25 = 10.8%
margin_rate_P50 = 13.8%
margin_rate_P75 = 18.4%
margin_rate_P90 = 25.1%
negative_margin_ratio ≈ 1.4%
extreme_margin_ratio ≈ 2.1%  # |rate| > 100%
```

极端毛利率主要由 GMV 极小造成 Tiny Denominator（极小分母）失真。

### 8.1 极端率的注意力规则

人工确认：

- 正向和负向极端毛利率都可以提醒；
- 不能只看率的绝对值，必须结合业务规模 / 金额影响；
- 负向异常需要防微杜渐；
- 但极小规模的 `-500%` 不应压过大规模、真实影响更大的 `-20%`。

因此：

> **Abnormality（异常程度） ≠ Materiality（重大性）。**

Attention 层必须至少结合：

```text
Rate Extremeness
×
Scale / Absolute Effect
×
Direction（同等情况下负向可适度提高关注）
```

Golden Scenario 应至少包含：

```text
A: 极小 GMV + 极高正毛利率 + 极小绝对影响
B: 极小 GMV + 极低负毛利率 + 极小绝对影响
C: 正常规模 + 中度负毛利率 + 较大绝对影响
```

验证系统不会把“比例夸张”与“经营影响重大”混为一谈。

---

## 9. Period State 的现实语义

上一轮内部统计按“门店 × 月份”而非标准 Atomic Grain 计算，结果仅作为现实存在性参考，**不得直接作为 Atomic Generator Probability**：

```text
STANDARD_ratio = 76.7%  (3178/4142)
NET_ZERO_PRESENT_ratio = 3.0%  (125/4142)
ZERO_GMV_NONZERO_GP_ratio = 5.6%  (231/4142)
NEGATIVE_GMV_ratio = 0.7%  (30/4142)
INVALID_OR_MISSING_ratio = 0%
ABSENT_ratio = 14.0%  (578/4142)
```

需要注意：这些比例的统计 Grain 与 Production Atomic Grain 不一致，因此未来若要作为 Generator 概率参数，必须按 `月份 × 门店 × 渠道` 重算。

### 9.1 NET_ZERO_PRESENT

人工确认：`GMV = 0, GP = 0, key_present = true` 背后都可能存在：

- 纯占位记录；
- 有销售 / 退货等正负冲销后归零；
- 没有业绩，但发生费用或其他经营活动；
- 其他合法业务过程。

因此：

> `NET_ZERO_PRESENT` 只描述当前指标域的数值状态，不等于“整个经营活动不存在”。

### 9.2 ZERO_GMV_NONZERO_GP

现实画像：

```text
GMV = 0 的门店月 ≈ 10.0%
GMV = 0 且 GP != 0 ≈ 6.5%
```

人工确认：这是合法经营事实，常见来源包括租赁、服务、广告等非商品 GMV 毛利，不应默认判成数据错误。

现实中这些 GP 的典型金额很小，中位数量级约为正常门店月 GP 中位数的万分之一。

因此 Generator 不应只模拟：

```text
occurrence_rate（发生频率）
```

还必须模拟：

```text
effect_size_distribution（发生时的典型影响规模）
```

这是 Golden Generator 的正式设计原则：

> **异常 / 非标准状态不仅要模拟“出现多少次”，还要模拟“出现时通常有多大”。**

### 9.3 NEGATIVE_GMV

现实画像：

```text
GMV < 0 的门店月 ≈ 0.8%
```

人工确认：负 GMV 基本可理解为退货、冲销、跨期调整等真实负业务记录，而不是数据错误。

底层机器层仍只保留机械事实：

```text
GMV < 0
```

不得未经 Evidence 自动把某一条具体记录解释为“退货”或“冲销”。

### 9.4 REALITY PROFILE 不修改状态机定义

这些现实解释不能改变 Production State Contract：

- `ABSENT`
- `STANDARD`
- `NET_ZERO_PRESENT`
- `ZERO_GMV_NONZERO_GP`
- `NEGATIVE_GMV`
- `INVALID_OR_MISSING`
- `OTHER_NONSTANDARD`

Reality Profile 只提供这些状态在真实业务中的可能背景和生成形状。

---

## 10. Base → Current 迁移：现有统计仅作参考

上一轮 2025-07 → 2026-07 统计：

```text
continuing_standard_ratio ≈ 84%  (146/174)
entry_ratio ≈ 7%  (13/188)
exit_ratio ≈ 9%  (17/191)
standard_to_nonstandard_ratio ≈ 4.6%  (8/174)
nonstandard_to_standard_ratio ≈ 4.6%  (8/174)
```

注意：这些比例使用了不同分母，因此**不能直接作为一个 Transition Probability Matrix**。

正式 Generator 若需要迁移概率，必须使用统一全集：

```text
Keys_Base ∪ Keys_Current
```

通过 Full Outer Join 构造完整状态迁移矩阵，并要求全部迁移比例在同一 Universe 上闭合到 100%。

---

## 11. 渠道毛利率结构

人工确认：渠道毛利率存在总体倾向，但不存在硬排序。

一般现实倾向：

```text
地采毛利率 > 集采毛利率 > 万家毛利率
```

但存在反例，例如某些门店 / 月份：

```text
集采毛利率 > 地采毛利率
```

因此 Generator 应使用**分布均值 / 中心位置不同但大量重叠**的方式生成，而不能使用不等式硬约束。

`集采 + 万家 = 大集采` 只用于派生分析口径，不改变 Atomic Channel。

---

## 12. Internal Offset：真实经营中的常态结构

人工确认：

> 总体变化不大、内部大量门店 / 渠道一涨一跌并互相抵消的情况非常常见。

其现实机制可能包括：

- 某条渠道路径未达到目标，其他渠道补偿；
- 某些门店下降，其他门店补偿；
- 关闭旧店、开启新店，并存在一段新旧店同时存续期；
- 其他资源 / 经营路径迁移。

这种现象可用“失之东隅，收之桑榆”作为业务直觉描述，但算法层不得未经 Evidence 自动推断为“考核压力导致补偿”。

### 12.1 两种 Offset 要区分

```text
Random Offset
= 子项变化碰巧正负抵消

Compensatory Migration
= 一个路径失速后，其他路径出现补偿性增长
```

仅靠结果数据，算法最多识别：

- Gross Movement 很大；
- Net Movement 很小；
- 存在结构迁移 / 抵消；

不能仅凭数学结果确认补偿行为的具体动机。

### 12.2 Golden Generator 的要求

正常 Golden Scenario 本身就应包含一定程度的内部抵消，而不是把 HIGH_OFFSET 只设计成极端异常。

另外单独保留：

```text
HIGH_OFFSET
```

用于攻击：

```text
Parent 净变化很小
但内部 Gross Movement 巨大
```

的场景。

---

## 13. 确认时点错配：本次只保留边界，不建模

现实事实：

- 当前 GMV 是出库口径，而非消费者成交口径；
- 发货本身可能滞后于成交；
- 后台返利确认也可能滞后，且并非固定滞后一个月；
- 因此单月 GMV 与毛利率变化可能受到 Recognition Lag（确认时点错配）影响。

但人工确认：**这不是当前单店损益 Golden Scenario 的重点。**

当前只冻结解释边界：

> 单月毛利率变化属于 Fact / Mathematical WHY，不能仅凭单月数据直接推出“返利能力下降”“以价换量”等业务因果。

返利滞后、品牌 / 品类 / 供应商拆解应留到未来“品毛表”事实域单独建模。

---

## 14. Control Total：官方考核报表优先

人工确认：提供给分析系统的底表最终一定会与官方考核报表总数对上。

因此正式运行的 Data Gate 应包含：

```text
Raw / Canonical Total
→ Reconciliation
→ Official Control Total
```

若不闭合，应优先检查：

- 特殊行被遗漏；
- 过滤条件；
- Grain / Aggregation；
- 符号处理；
- 字段映射；
- 数据版本；

而不是直接进入经营解释。

---

## 15. 当前 Golden Generator 可采用的现实参数摘要

```text
[GOLDEN_REALITY_PROFILE_EAST_CHINA_V1]

organization:
  division_count = 11
  store_count = 218
  channel_count = 4
  store_type_count = 8

atomic_grain:
  month × store × channel
  store_type = attribute

store_structure:
  store_size_skewness = HIGH
  store_weight_concentration = LOW_TO_MEDIUM
  max_vs_median = 21.5x
  top1_share = 5.5%
  top3_share = 13.2%
  top5_share = 18.8%

  store_type_affects_scale = YES
  within_store_type_dispersion = HIGH

division_structure:
  division_skewness = LOW_TO_MEDIUM
  max_vs_median = 1.82x
  top1_share = 15.2%
  top3_share = 40.8%

channel:
  atomic_channels = [地采, 集采, 万家, 星选]
  derived_group: 大集采 = 集采 + 万家
  overall_share = [71.0%, 15.4%, 11.9%, 1.7%]
  channel_mix_heterogeneity = HIGH
  short_term_inertia = HIGH
  long_term_structural_migration = ALLOWED
  store_type_channel_correlation = SOFT

time:
  base_months = 12
  current_months = 7
  seasonality = HIGH
  store_entry = YES
  store_exit = YES
  old_new_store_overlap = ALLOWED
  pre_open_sales = ALLOWED
  post_close_sales = ALLOWED
  ytd_vs_single_month_reversal = COMMON

margin:
  P10 = 8.2%
  P25 = 10.8%
  P50 = 13.8%
  P75 = 18.4%
  P90 = 25.1%
  negative_margin_frequency ≈ 1.4%
  tiny_denominator_extreme_rate_frequency ≈ 2.1%
  materiality_must_consider_scale = YES

state_semantics:
  ABSENT = reliable_no_business_at_atomic_grain
  NET_ZERO_PRESENT = valid_present_state_not_equal_absent
  ZERO_GMV_NONZERO_GP = legal_business_fact
  NEGATIVE_GMV = legal_nonstandard_business_fact

special_rows:
  no_store_id_rows_exist = YES
  typical_materiality = LOW
  must_reconcile_to_parent_total = YES

movement:
  internal_offset = COMMON
  compensatory_migration = POSSIBLE
  cause_requires_evidence = YES

recognition_lag:
  exists = YES
  model_in_current_mvp = NO
```

---

## 16. 当前最值得 Golden Scenario 保留的现实结构特征

按“最容易击穿经营分析算法 / AI 语义”的标准，当前优先保留：

1. **门店规模高度偏斜，但总体权重并非被一两家门店完全垄断。**
2. **渠道结构短期稳定、长期可因制度 / 政策发生明显迁移，且 Atomic Entry 不等于新店开业。**
3. **`ABSENT / NET_ZERO_PRESENT / ZERO_GMV_NONZERO_GP / NEGATIVE_GMV` 都可能是真实合法状态，数值状态不能越权翻译成业务因果。**
4. **总体小变化下内部大幅正负抵消非常常见，必须同时看 Gross Movement 与 Net Movement。**
5. **异常率必须结合规模判断重大性；Tiny Denominator 可以制造极端毛利率，但并不天然代表重大经营影响。**

额外必须保留的边界：

- 少量无门店 ID 特殊行必须参与总数闭合；
- 新旧门店可以存在并行存续期；
- 单月毛利不能自动解释返利 / 促销因果；
- 大集采是派生口径，不能制造第五个 Atomic Channel。

---

## 17. 尚未冻结为 Generator 概率的内容

以下内容仍需在真正使用前单独计算或人工设计，不能直接从当前粗画像复制：

1. Atomic Grain（月份 × 门店 × 渠道）下的完整 Period State 分布；
2. 统一 `Keys_Base ∪ Keys_Current` Universe 下的 Transition Matrix；
3. Special Rows 对 GMV / GP 的精确占比；
4. 正式 `Offset = 1 - |ΣΔ| / Σ|Δ|` 的门店 / 渠道分布；
5. Tiny Denominator 的 GMV / GP 相对正常原子规模分布；
6. 店型 × 渠道 × 毛利率的具体联合分布。

这些缺口**不阻塞 GS-001 Normal Baseline**。GS-001 可以使用现实范围 + 人工可控规则生成，再由独立 Oracle 给出 Expected Result。

---

## 18. 对 GS-001 的直接约束

首个 `GS-001 NORMAL_BASELINE` 不追求极端变态，而应模拟“现实世界里普通但不干净的月份”：

- 组织规模接近真实形状，但可按测试运行成本缩放；
- 4 个 Atomic Channel；
- 店型是属性；
- 门店规模长尾；
- 渠道结构门店间差异明显；
- 有季节性和正常月度波动；
- 有少量 Atomic Entry / Exit，但不自动解释为门店开关；
- 有少量合法非标准状态；
- 有少量 Special Rows；
- 有常态内部抵消；
- 有少量 Tiny Denominator，但不能主导 Parent 结果；
- 最终 Synthetic Raw Data 必须与 Synthetic Official Control Total 闭合。

成功标准不是“生成一张漂亮数据表”，而是：

> **在正常、复杂、但没有故意塞入极端陷阱的现实形状数据上，Production Baseline 能完整完成 Data Gate → WHAT → WHERE → Mathematical WHY → Boundary / Attention，并与独立 Oracle 结果一致。**

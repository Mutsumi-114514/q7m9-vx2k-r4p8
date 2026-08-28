# Test Sample Specification V0.1：测试样本与记录规范

> 定位：A 阶段 Deterministic Test Suite（确定性测试集）、Scenario Test（完整业务场景测试）、Property Test（性质 / 随机测试）和 Regression Test（回归测试）的统一样本合同。  
> 当前适用范围：单店表 MVP，当前原子颗粒度为 `时间 × 门店 × 渠道`。  
> 目标：不以“跑了多少随机数”代替覆盖率，而是同时保证 **可复现、状态覆盖、结构覆盖、业务形状覆盖和历史缺陷永久回归**。

---

## 1. 总原则

测试体系采用四层结构：

```text
L1 Canonical / Boundary / Regression
   微型确定性案例
        ↓
L2 Scenario Test
   接近真实经营组织形状的完整合成数据集
        ↓
L3 Property / Adversarial Random Test
   大规模可复现随机与对抗性质测试
        ↓
L4 Multi-Agent Adversarial Review
   不同 AI 独立创造新反例
```

核心原则：

> **小样本证明规则；大场景证明系统；随机样本寻找未知反例；被确认的反例永久转化为回归测试。**

以及：

> **Sample Count（样本量）不能替代 Coverage（覆盖率）。**

100 万个重复的 `STANDARD → STANDARD` 随机样本，不如一个能击穿状态机的 `NET_ZERO_PRESENT ≠ ABSENT` 反例有价值。

---

## 2. 测试证据必须留下

正式测试不能只在对话里报告“已跑 X 万组”。

所有正式测试至少必须能够回答：

- 测了哪个 Baseline；
- 测试输入是什么；
- 期望结果是什么；
- 实际结果是什么；
- 判定条件是什么；
- 如果是随机测试，怎样完全复现；
- 如果失败，失败样本在哪里；
- 该失败是否已进入永久 Regression。

正式记录属于 **Test Evidence（测试证据）**。

探索性临时测试如果没有保留 Seed / Generator / Case，则只能称为 Exploratory Test（探索性测试），不能作为正式 Baseline 通过证据。

---

## 3. 单条确定性测试记录合同

每个 Canonical / Boundary / Regression Case 至少包含：

```text
case_id
case_name
case_category
purpose

baseline_commit
algorithm_version
data_contract_version
metric_definition_version

grain
unit_of_measure

base_input
current_input

expected_base_state
expected_current_state
expected_transition_type
expected_components
expected_boundary_status
expected_invariants

failure_condition
tolerance
source
```

其中：

### `case_category`

建议 Machine Code：

```text
CANONICAL
BOUNDARY
REGRESSION
SCENARIO_ASSERTION
```

### `source`

至少区分：

```text
HAND_AUTHORED
REPOSITORY_AUDIT
REVIEWER_GPT
REVIEWER_DOUBAO
REVIEWER_QWEN
RANDOM_DISCOVERY
REAL_WORLD_ANONYMIZED
```

### `expected_invariants`

只声明当前算法适用的不变量，例如：

```text
Closure = PASS
Time Reversal = PASS
Symmetry = PASS | N/A
Order Independence = PASS | N/A
Boundary Routing = PASS
Semantic Validity = PASS
```

不得把不适用的不变量强制写成 PASS。

---

## 4. 当前 Period State 覆盖

当前 Machine State：

| Machine Code | 中文标准名 | 当前测试方式 |
|---|---|---|
| `ABSENT` | 无原子记录 | 数值 + Presence |
| `STANDARD` | 标准业务状态 | 数值 |
| `NET_ZERO_PRESENT` | 有记录但净额归零 | 数值 + Raw Row 抵消 |
| `ZERO_GMV_NONZERO_GP` | 零GMV非零毛利 | 数值 |
| `NEGATIVE_GMV` | 负GMV状态 | 数值 |
| `INVALID_OR_MISSING` | 数据缺失或无效 | Schema / Null |
| `OTHER_NONSTANDARD` | 其他非标准状态 | 合同测试；只有存在额外机械判定规则时才能实例化 |

### 4.1 状态迁移覆盖

正式 V0.1 Coverage Matrix 以：

```text
Base State × Current State
```

为基础。

理论位置：

```text
7 × 7 = 49
```

其中：

- 可执行定义明确的状态组合必须有对应路由测试；
- `ABSENT → ABSENT` 应验证不会从 Full Outer Join 生成虚假 Atom；
- `OTHER_NONSTANDARD` 没有具体机械判定规则时，只测试“不得由 AI 自行生成”；
- `ABSENT → STANDARD` 才允许自动 `PURE_ENTRY`；
- `STANDARD → ABSENT` 才允许自动 `PURE_EXIT`；
- `NET_ZERO_PRESENT` 不得冒充 `ABSENT`。

---

## 5. L1：微型确定性测试规模

V0.1 目标约 **100～120 个永久案例**，初始规划约 110 个。

建议分配：

| 测试组 | 目标案例数 | 核心目的 |
|---|---:|---|
| State / Transition | 约49 | 状态、Presence、路由 |
| Bennett / GP Amount | 约12 | Scale / Rate / Entry / Exit / Non-standard |
| Parent Rate / Mix / Bridge | 约20 | 三层 Bridge、Mix / Rate、Membership |
| Roll-up / View | 约10 | Atomic vs Parent、Derived Ratio |
| YTD / Time | 约6 | 时间聚合与重新分解 |
| Type / Unit / Schema | 约5 | 单位、类型、版本、Null |
| Known Regression | 起步约8 | 已知历史缺陷永久防回归 |

案例数是 V0.1 目标，不是为了凑数；如果一个状态需要多个最小反例，允许增加。

---

## 6. 已知必须永久保留的 Regression Case

### 6.1 Net Zero Is Not Absent

Base 原始记录：

```text
row1: GMV +100, GP +20
row2: GMV -100, GP -20
```

聚合：

```text
key_present = true
GMV = 0
GP = 0
```

Current：

```text
key_present = true
GMV = 50
GP = 10
```

期望：

```text
base_state = NET_ZERO_PRESENT
current_state = STANDARD
transition_type != PURE_ENTRY
```

### 6.2 Entry / Exit Time Reversal Mapping

Forward：

```text
ABSENT → STANDARD
Entry = +X
```

Reverse：

```text
STANDARD → ABSENT
Exit = -X
```

必须满足：

```text
reverse.exit = -forward.entry
reverse.entry = -forward.exit
```

### 6.3 Parent Bennett Zero-denominator Gate

任一期间 Parent GMV 为 0 且 Parent Rate 无定义时：

```text
Parent Bennett = N/A
boundary_status = BOUNDARY_STOP
```

禁止硬算率。

### 6.4 Atomic GP Unified Closure

对任何数据有效原子：

```text
atomic_gp_effect_total = current_gp - base_gp
```

不同 Transition 的组件必须闭合到该值；全 Parent：

```text
Σ atomic_gp_effect_total = Δ Parent GP
```

---

## 7. L2：真实形状 Scenario Test

微型单元测试不足以验证真实经营组织中：

- 大店压过小店；
- 多渠道结构差异；
- 店型结构迁移；
- 12 个月季节性；
- Entry / Exit 跨月出现；
- 大量内部对冲；
- YTD 与单月结论不同；

等组合问题。

因此 V0.1 固定增加 **Scenario Test（完整场景测试）**。

### 7.1 Reference Scenario Shape V1

当前参考完整业务形状：

```text
门店数          = 20
渠道数          = 4
Current Months  = 12
Base Months     = 12
门店类型        = 4
```

当前原子颗粒度仍是：

```text
时间 × 门店 × 渠道
```

因此完整月度原子规模约：

```text
20 × 4 × (12 + 12)
= 1,920 monthly atoms
```

门店类型是门店 Attribute（属性），不是独立原子轴，因此不额外乘 4。

### 7.2 Raw Row Multiplicity

数据合同允许一个 `月份 × 门店 × 渠道 × 属性` 下存在多条合法记录。

Scenario Generator 应允许每个聚合原子由多条底层 Raw Row 构成，以测试：

- 正负抵消；
- 不自动去重；
- Presence 与净额分离；
- 聚合前后恒等式。

如果平均每个原子产生 5～20 条 Raw Row，一套场景可自然达到约 1 万～4 万底层记录。

---

## 8. Scenario 中必须有非均匀经营结构

不能让 20 家门店全部 GMV=100。

### 8.1 门店大小

至少覆盖：

```text
超大店 / 大店 / 中店 / 小店
```

门店规模分布必须不均匀，并至少存在：

- 单一巨店占 Parent 很高权重；
- 多数小店合计占比较高；
- 相对均匀结构；

等不同 Concentration（集中度）场景。

测试范围仅用于合成数据，不代表真实业务定义。

### 8.2 门店类型

Reference Scenario 包含 4 种店型。

店型是门店属性，可用于 Roll-up / Re-decomposition；不得作为新的独立事实轴重复制造数据。

### 8.3 渠道结构

不同门店的 4 个渠道占比不得完全一致。

Scenario 应包含：

- 单渠道高度集中门店；
- 多渠道均衡门店；
- 新增渠道；
- 退出渠道；
- 渠道结构显著迁移。

### 8.4 时间结构

24 个月不能全部平稳。

至少包含：

- 季节性；
- 趋势增长 / 下降；
- 单月突增 / 突降；
- 新店 / 关店；
- 新渠道 / 渠道退出；
- 非标准状态集中月；
- 强对冲月；
- YTD 与单月方向不同。

---

## 9. V0.1 固定 Scenario Dataset

建议起步冻结 **8～10 套**完整 Scenario，每套约 1,920 个 Monthly Atom。

初始建议 10 套：

| Scenario Code | 中文 | 主要攻击点 |
|---|---|---|
| `NORMAL_BASELINE` | 正常经营基线 | 全链路正常闭合 |
| `SIZE_CONCENTRATION` | 门店规模高度集中 | 巨店权重、Parent 影响 |
| `STRUCTURAL_MIGRATION` | 结构迁移 | 高毛利 → 低毛利结构迁移 |
| `ENTRY_EXIT_WAVE` | 大量新增退出 | 店 / 渠道 Entry / Exit |
| `NONSTANDARD_HEAVY` | 非标准状态密集 | Zero-GMV GP、Negative GMV、Net Zero |
| `HIGH_OFFSET` | 高对冲 | 净额很小、内部运动巨大 |
| `FULL_REPLACEMENT` | 业务集合完全替换 | Continuing Set 为空 |
| `SEASONAL_YTD` | 强季节性累计 | 月度 vs YTD 重算 |
| `HIERARCHY_CHANGE` | 属性 / 组织变化 | 当期有效店型 / 归属 |
| `MIX_RATE_PARADOX` | 结构反直觉 | 子组率改善但 Parent 率恶化等 |

10 套场景约：

```text
19,200 monthly atoms
```

底层 Raw Rows 可达到十万级以上，但不要求为了行数而人为膨胀。

---

## 10. Golden Synthetic Dataset：合成黄金数据集

L2 Scenario Test 不应只有“像真实数据”的随机大表，还应包含 **答案已知、可以黑盒验收的合成黄金数据集**。

它解决三个问题：

1. 真实业务数据最接近生产，但正确分析答案通常需要人工重新复核；
2. 微型确定性案例答案明确，但离真实组织形状较远；
3. Golden Synthetic Dataset 同时保留 **真实结构复杂度 + 已知标准答案 + 无真实经营数据泄露**。

核心定义：

> **Golden Synthetic Dataset = 使用真实业务结构参数构造、但经营数值完全合成，并拥有冻结 Scenario Truth 与 Expected Result 的完整测试数据集。**

### 10.1 三类 Golden Scenario

| 类型 | Machine Code | 目的 |
|---|---|---|
| Designed Golden Scenario | `DESIGNED_GOLDEN` | 人工设计明确业务现象，验证系统能否识别指定结构与陷阱 |
| Generated Golden Scenario | `GENERATED_GOLDEN` | 根据业务世界规则自动生成底层数据，再由 Oracle 计算标准答案 |
| Adversarial Golden Scenario | `ADVERSARIAL_GOLDEN` | 专门构造反直觉、强对冲、Simpson 类结构、Entry / Exit / Boundary 等陷阱 |

三类可以共享同一 Reference Scenario Shape，但生成逻辑和测试目的必须记录。

### 10.2 Scenario Truth 与 Raw Dataset 必须分离

每个 Golden Scenario 至少分成：

```text
Scenario Profile
    真实形状参数：门店数、渠道数、店型、月份、规模分布等

Scenario Truth
    这个虚拟经营世界被设计成“真正发生了什么”

Raw Synthetic Dataset
    根据 Profile + Truth 生成的底层测试表

Expected Result
    Oracle Engine 从 Raw Dataset 独立计算得到的标准数学结果
```

Scenario Truth 描述业务机制，而不是直接填写每一个算法组件的目标值。

例如可以规定：

```text
大店整体规模下降
小店整体增长
低毛利渠道份额上升
Continuing 单元毛利率小幅改善
存在新店和退出渠道
```

然后由 Generator 生成底层 GMV / GP，再由 Oracle 计算实际的 Scale / Rate / Mix / Entry / Exit 等标准答案。

### 10.3 禁止为了目标算法结果硬凑数据

不推荐：

```text
先规定 Mix = -0.80pp
↓
反复调整原始数字直到刚好等于 -0.80pp
```

这种做法容易造成 Test Overfitting（测试过拟合）：测试数据无意中迎合当前算法，而不是独立验证算法。

正式顺序应为：

```text
定义业务世界规则
↓
生成 Raw Synthetic Dataset
↓
独立 Oracle Engine 计算
↓
冻结 Expected Result
```

如果是 Designed / Adversarial Golden Scenario，可以人工规定方向性和结构性真相，例如：

- Parent 总 GMV 基本不变；
- 高毛利单元份额下降；
- 各子组自身毛利率改善；
- Parent 毛利率最终恶化；

但最终精确的 pp / 金额分解结果仍应由 Oracle 从底层数据计算，而不是人为倒填。

### 10.4 Oracle Engine 必须与被测 Skill 分离

Golden Dataset 的 Expected Result 必须由 **Oracle（标准答案计算器）**产生。

原则：

> **被测对象不能自己给自己出标准答案。**

Oracle 至少要求：

```text
oracle_version
baseline_commit
data_contract_version
input_fingerprint
expected_result_hash
```

在工程实现允许时，Oracle 应尽量采用：

- 独立实现路径；
- 简单直接的参考公式；
- 明确的中间结果；
- 可人工抽查的最小核心 Case；

以降低“Production 与 Oracle 共享同一个 Bug”的风险。

### 10.5 Blind Black-box Acceptance Test

Golden Dataset 特别适用于正式 Blind Review / Skill Acceptance。

内部 Test Evidence 保存：

```text
Raw Dataset
Scenario Truth
Expected Result
Generator Version / Seed
Oracle Version
Baseline
```

Blind Reviewer / 被测 Skill 默认只获得：

```text
Raw Dataset
+
本轮允许读取的 Skill / Spec
```

不得提前看到：

```text
Scenario Truth
Expected Result
```

运行完成后，再由 Comparator 对比：

```text
Expected vs Actual
```

至少可比较：

- Parent 总量与同比 / 差额；
- WHERE 排序与关键拉动 / 拖累对象；
- Parent GP Scale / Rate；
- Parent Margin Non-standard / Exit / Mix / Rate / Entry；
- Boundary Routing；
- Atomic vs Parent View；
- Unit / Type；
- 必须停止的语义边界。

这样正式三方评审不仅能评价“说得是否像经营分析”，还可以执行 **Black-box Acceptance Test（黑盒验收测试）**。

### 10.6 真实业务参数可以进入 Generator，真实经营数据不必进入

Golden Scenario 可以使用真实环境的 **结构参数** 提高代表性，例如：

```text
门店数量
渠道数量
店型数量
Base / Current 月份数量
表头字段
宽表 / 长表形态
最细颗粒度
门店规模分布
典型渠道集中度
正常毛利率范围
季节性强弱
新店 / 关店 / 渠道增退频率
Raw Row Multiplicity
```

这些参数用于生成一个“像真实经营系统”的合成世界，但：

> **具体 GMV、毛利、费用、返利等经营数值仍然是合成值。**

测试资产不因追求真实形状而要求把真实经营数据写入 GitHub。

### 10.7 Golden Dataset 与真实业务数据职责不同

正式定位：

```text
Golden Synthetic Dataset
→ 验证算法正确性
→ 验证 Router / Boundary / Type
→ 验证 WHERE / Mathematical WHY
→ 验证 Skill 黑盒行为

Real Business Data
→ Reality Check（现实校验）
→ 验证合成世界是否遗漏真实数据状态、口径异常或业务特殊性
```

真实案例不承担第一层“证明算法公式正确”的主要责任。

Reality Check 发现的新状态或异常，如果被确认具有一般性：

```text
真实世界发现
↓
抽象为不可逆匿名化 / 合成反例
↓
加入 Canonical / Boundary / Regression / Golden Scenario
```

这样真实数据只负责发现测试世界的盲区，不直接成为永久敏感测试资产。

### 10.8 Golden Scenario 的最小记录合同

每套 Golden Scenario 至少保存：

```text
scenario_id
scenario_type
scenario_profile_version
scenario_truth_version

generator_name
generator_version
random_seed                    # 若适用
oracle_name
oracle_version

baseline_commit
data_contract_version
metric_definition_version

raw_dataset_ref
input_fingerprint
expected_result_ref
expected_result_hash

blind_visibility
coverage_tags
```

其中：

```text
blind_visibility
```

至少区分：

```text
INTERNAL_TRUTH_ONLY
BLIND_REVIEW_INPUT
POST_REVIEW_REVEAL
```

避免在正式 Blind Review 前把标准答案意外暴露给 Reviewer。

---

## 11. Cross-factor Coverage（交叉因素覆盖）

真实系统风险来自多因素交叉，而不是单变量。

当前至少考虑以下维度：

1. Base State；
2. Current State；
3. GMV 数值状态：正 / 0 / 负 / 极小 / 极大；
4. GP 数值状态：正 / 0 / 负；
5. Margin 状态：正 / 0 / 负 / 极端；
6. Parent Membership：Continuing / Entry / Exit / Mixed / Replacement；
7. View：Atomic Attribution / Parent Re-decomposition；
8. Time Direction：Forward / Reverse；
9. Aggregation：Atomic / Store / Channel / Store Type / Parent / YTD；
10. Movement Pattern：同向 / 反向 / 强对冲 / 完全对冲；
11. Store Size / Weight Concentration；
12. Channel Mix Concentration。

不对上述维度做完整 Cartesian Product（笛卡尔积），否则会产生大量无意义组合。

V0.1 原则：

> **State Transition 全覆盖；其他维度采用 Pairwise（两两覆盖）+ Adversarial（对抗场景）覆盖。**

如果未来发现某个三因素组合才能触发 Bug，该组合永久进入 Regression Test。

---

## 12. L3：Property / Adversarial Random Test

V0.1 建议起步约 **250,000 Parent Cases**，不是硬性长期上限。

至少分为 5 个 Generator：

```text
G1 Standard Bennett               ~50,000
G2 Continuing Mix / Rate          ~50,000
G3 Entry / Exit / Membership      ~50,000
G4 Non-standard / Zero / Negative ~50,000
G5 Parent / Roll-up / Offset      ~50,000
```

每个随机 Parent 建议包含：

```text
1～20 atoms
```

随机测试必须使用 Stratified Random（分层随机），故意提高危险边界出现概率，而不是按照真实业务频率采样。

测试分布不是业务预测分布。

---

## 13. 随机测试记录合同

每次随机测试 Run 至少保存：

```text
run_id
baseline_commit
generator_name
generator_version
random_seed
sample_count
parameter_space
invariant_set
started_at
finished_at

pass_count
fail_count
failed_case_ids
max_closure_error
```

`parameter_space` 至少说明：

- Atom 数量范围；
- 各 Period State 生成概率；
- GMV / GP / Rate 数值范围；
- 是否允许负毛利；
- 是否允许 Negative GMV；
- 是否允许 Zero denominator；
- 是否允许 Full Membership Replacement；
- 门店 / 渠道权重分布；
- 是否生成 Reverse Pair。

通过样本无需全部永久保存为巨大文件；只要 Seed + Generator Version 足够完全复现即可。

> **任何失败样本必须保存完整输入、实际输出、预期性质和 Case ID。**

---

## 14. Coverage Matrix 必须成为正式测试产物

不能只写：

> “这些都测试过。”

至少维护类似：

| 场景 | Deterministic | Reversal | Scenario | Golden | Random | Regression |
|---|---:|---:|---:|---:|---:|---:|
| Pure Scale | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Pure Rate | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Pure Entry | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Pure Exit | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Net Zero ≠ Absent | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Zero GMV / Non-zero GP | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Negative GMV | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Full Replacement | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Parent denominator 0 | ✓ | N/A / Mapping | ✓ | ✓ | ✓ | ✓ |
| Strong Offset | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| YTD | ✓ | 按适用规则 | ✓ | ✓ | ✓ | — |
| Atomic → Parent View | ✓ | N/A | ✓ | ✓ | ✓ | ✓ |

未来测试是否“足够”，优先看 Coverage Matrix，而不是只看总 Case 数。

---

## 15. L4：Multi-Agent 新反例进入永久测试集

GPT / 豆包 / 千问等 Reviewer 在正式 Blind Review 中必须独立创造测试。

任何 Reviewer Finding 经 Human Adjudication：

```text
ACCEPT
```

则：

> **Every Accepted Counterexample Becomes a Regression Test。**

流程：

```text
Reviewer 发现反例
↓
Cross Review / Deterministic Runner 验证
↓
Human Adjudication = ACCEPT
↓
分配永久 case_id
↓
加入 Regression Suite
↓
以后任何 Baseline 修改必须继续通过
```

测试集因此只会因新知识增长，不应因为 Bug 已修复而删除对应回归案例。

---

## 16. 当前 MVP 与未来扩展边界

当前 V0.1 Scenario 固定围绕：

```text
时间 × 门店 × 渠道
```

以及门店类型、门店大小等 Attribute / Grouping。

暂不为了未来可能增加的：

- 大区；
- 分部；
- 品类1 / 品类2 / 品类3；
- 品牌 / SKU；
- 供应商；
- 费用项目；
- 后台返利类型；

提前扩张当前测试数据的笛卡尔规模。

但测试结构必须保留：

```text
grain
dimensions
attributes
hierarchy
metric
aggregation_semantics
unit_of_measure
```

未来增加维度时，优先扩展 Generator / Scenario Profile / Coverage Matrix，不重写测试治理框架。

> **MVP 先验证框架；新维度进入时复用同一 Test Contract，再补该维度特有的数学语义和边界测试。**

---

## 17. V0.1 通过门槛

正式冻结下一版 Baseline 前，最低要求：

1. L1 核心确定性案例全部 PASS；
2. 49 个 State Transition 位置已完成合法 / 非法路由覆盖说明；
3. 至少 8 套完整 Scenario Dataset 通过核心闭合与路由检查，其中至少包含一套 Golden Synthetic Dataset 黑盒验收；
4. Golden Dataset 的 Scenario Truth / Expected Result 与 Blind Reviewer 输入完成权限分离；
5. Property Test 完成目标规模并保留可复现 Run Record；
6. 所有失败案例已保存；
7. 所有已裁决历史 Bug 均有 Regression Test；
8. Coverage Matrix 无未解释的高风险空白；
9. 测试使用未舍入值执行 Closure；
10. 任何 Invariant Failure 均阻止进入 AI Interpretation；
11. 测试证据绑定明确的 Baseline Commit。

通过门槛达到后，才冻结正式三方 Blind Review 使用的 Baseline SHA。

---

## 18. 最终原则

> **测试不是证明“我们生成了很多数字”，而是证明系统在已知规则、边界状态、真实组织形状和大规模未知组合下都难以被轻易击穿。**

Golden Synthetic Dataset 进一步要求：

> **真实结构可以借鉴，真实数值不必进入；先定义虚拟世界，再生成数据，再由独立 Oracle 冻结答案；Blind Reviewer 只看考卷，不看答案。**

V0.1 的目标不是建立无限大的测试宇宙，而是建立一个能够持续吸收新反例、可复现、可审计、可黑盒验收、可随维度扩展的测试体系。

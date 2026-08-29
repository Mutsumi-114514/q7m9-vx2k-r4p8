# Operating Analysis Skill Roadmap

> 状态：`CURRENT PROJECT ROADMAP / NOT IMPLEMENTATION SOURCE`  
> 日期：2026-08-29  
> 用途：记录从 Baseline V1 收口、单店表 Runtime Skill、真实业务验证，到品毛表、多表联合分析、方法论 Skill、审核 Skill 与长期维护的项目路线。  
> 重要边界：本文描述未来路线，不代表对应能力已经实现；任何 Production Methodology 仍以 Canonical Contract 为准。

---

## 0. 现在：先把 Baseline V1 收口

当前阶段不继续扩张算法库。

先完成三类已经经过多轮审计、且仍值得修复的 Contract Closure：

1. **Test Governance**：区分 Methodology Contract Freeze 与未来 Executable Implementation Acceptance；
2. **Comparison Window Granularity**：比较窗口不能比 Canonical Input 的时间粒度更细；
3. **Registered Cohort Contract**：只有 membership 能唯一解析、来源与版本可追踪的 Cohort 才进入 Production V1。

修改完成后，再做一次**全仓库审核**。原因很简单：

> 局部修正正确，不代表整个仓库叠加以后仍然正确。

全局审核重点不是继续发明功能，而是确认：

- Source of Truth 是否仍然唯一；
- 新规则是否与旧规则冲突；
- README / Handoff / Supporting Contract 是否出现版本漂移；
- 数学、Population、Boundary、Materiality、Cross-View、Testing 是否还能构成同一个系统；
- 是否存在“每一段都合理，但两段放在一起会给两个答案”的情况。

通过后再 Freeze Baseline V1。

---

## 1. 第一落地点：单店表 Runtime Skill

第一版真正可用的 Skill 不追求万能。

先稳定回答真实工作中最常见的问题，例如：

> “南京这个月综合毛利为什么同比下降？”

目标链路：

```text
自然语言问题
→ Metric / Scope / Population / Window 解析
→ 数据合同与控制数校验
→ 确定性 Math Engine
→ 多维度 WHAT / WHERE
→ Materiality / Collective Materiality
→ Cross-View Guard
→ Mathematical WHY
→ Evidence Questions
```

第一版最重要的不是报告写得多漂亮，而是：

> **同一个业务问题，不能因为换一个模型、换一句表达、换一个 Skill，就进入另一套数学世界。**

---

## 2. 真实工作成为主测试场

Runtime Skill 出来以后，项目的主要推进方式从“继续想象边界”切换为：

```text
真实问题
→ Skill 输出
→ 人工复核
→ Production Counterexample
→ 判断错误属于哪一层
→ 最小修复
→ Regression
```

错误至少区分：

```text
Input / Mapping Bug
Scope / Population Bug
Contract Bug
Algorithm / Engine Bug
Boundary Bug
Explanation Bug
Attention / Ranking Bug
```

如果只是实现违反现有 Contract：

> 修实现，不改方法论。

如果真实反例证明 Contract 本身允许两个答案：

> Human Adjudication → Contract Version Change → Regression。

这样真实工作会逐渐变成系统最重要的训练场和质量来源。

---

## 3. 第二落地点：品毛表分析

品毛表不是重新发明一套经营分析方法。

它本质上可以理解为：

> **GMV + 拆到更细口径的毛利事实 + 更丰富的商品 / 供应商维度。**

相较单店表，主要新增的分析维度可能包括：

```text
品类1 / 品类2 / 品类3
品牌
SKU（若未来确有稳定口径）
供应商
采购组织 / 供应链相关属性
其他已确认商品属性
```

可以大量复用现有基础规则：

- Comparison Window；
- Comparable / Population；
- Additive Amount；
- Rate Recalculation；
- Bennett / Mix-Rate 在适用指标上的规则；
- Materiality；
- Collective Materiality；
- Cross-View Non-Additivity；
- Mathematical WHY 与现实因果边界。

真正需要新增的是**品毛表自己的 Data Contract、Atomic Grain、字段恒等式与维度 membership**。

原则：

> **复用方法，不偷换 Grain。**

品毛表可以比单店表更细，但不能因为两个系统都包含 GMV / 毛利，就假设两张表任意字段都可以直接 Join 或相加。

---

## 4. 第三个落地点：单店表 + 品毛表联合分析

这一步会带来真正的分析深度提升。

单店表更擅长回答：

> 哪个门店、渠道、经营环节最终出了问题？

品毛表更擅长回答：

> 商品结构、品类、品牌、供应商到底在哪里发生变化？

联合后理想链路可能是：

```text
综合毛利下降
→ 定位主要门店 / 渠道
→ 判断商品毛利还是费用 / 其他毛利主导
→ 若进入商品毛利
→ Drill-down 到品毛表
→ 品类 / 品牌 / 供应商
→ 形成更具体的 Mathematical WHY
→ 再进入现实 Evidence
```

这里最大的风险不是算法，而是**跨 Grain 解释**。

因此联合分析必须先定义：

- 两张表各自的 Source of Truth；
- 能否在某个共同控制数上勾稽；
- Drill-through 与 Join 的边界；
- 哪些结论只是两个 View 对同一现象的投影；
- 哪些指标只有单店表存在、哪些只有品毛表存在；
- 什么时候可以继续下钻，什么时候必须停在数据边界。

原则：

> **先证明两张表在当前问题上属于同一个可比世界，再联合解释。**

---

## 5. 从具体应用抽象出 Methodology Skill

当单店表与品毛表都经过真实使用后，可以把其中真正稳定、跨数据集复用的部分抽象出来。

目标不是“再做一个万能分析 Prompt”，而是形成一个更高层的：

> **Operating Analysis Methodology Skill**

它主要帮助回答：

```text
用户究竟在问什么？
目标指标的数学结构是什么？
Scope / Population / Window 应该怎样定义？
应该选择哪一个 Canonical Method？
哪些边界必须拒绝计算？
需要哪些 Control Total / Invariant？
Materiality 应该如何定义？
需要哪些 Regression / Golden Case？
数学结论能说到哪里，现实因果需要什么 Evidence？
```

它的价值不是直接替代具体 Runtime Engine，而是：

> **帮助一个新的经营分析问题快速获得一套可靠、可测试、可长期维护的方法。**

---

## 6. 把本项目的审核过程也做成 Skill

多轮 Blind Review 已经逐渐形成一套可复用的方法：

```text
从零阅读
→ 提出 Finding
→ 要求具体失败机制
→ Reviewer Self-Falsification
→ 区分 Contract Defect / Implementation Gap / Enhancement
→ Human Adjudication
→ Accepted Counterexample → Regression
→ Stop Rule / Freeze Readiness
```

未来可以将其沉淀为：

> **Methodology / Baseline Review Skill**

它不是自动替人“判 PASS”，而是帮助人更高质量地审核方法论或重量级 Skill。

核心纪律：

- Finding 不等于 Bug；
- HIGH 必须尽量给出 `合法输入 → 合规路径 / 未定义状态 → 不同结果`；
- 不按 Reviewer 数量投票；
- Reviewer 必须尝试推翻自己；
- 不能把“未来还能增强”冒充 Baseline Blocker；
- 已确认反例进入 Regression；
- 最终裁决仍由 Human Adjudication 完成。

这个 Skill 如果成熟，可以反过来审核未来的品毛表 Skill、联合分析 Skill、Methodology Skill 本身。

---

## 7. 再扩展算法，而不是先囤算法

现有 CFO Skill 生态已经证明：算法数量多不等于方法统一。

未来 Delta、PVM、LMDI、Chain、Haltiwanger、Shapley 等方法只有在真实业务问题需要时才加入。

每增加一种方法，先回答：

> **它解决了现有 Canonical Method 无法回答的什么问题？**

如果两个方法面对相似数学结构却回答不同问题，必须在 Contract 中把“问题定义”区分开，而不是简单写成“都可以”。

原则：

> **算法数量不是 KPI；语义唯一性比算法丰富度更重要。**

---

## 8. Attention：等“算得太多”真的成为生产问题再展开

当多维度计算稳定以后，系统很可能会从“找不到问题”进入另一个阶段：

> 找到的正确问题太多了。

这时再重新打开：

- Finding Consolidation；
- Cross-View 去重 / 关联；
- Gross / Net / Offset 的注意力策略；
- Recall → Rerank；
- L1 / L2 / L3 Progressive Disclosure；
- Main WHY 排序。

Attention 只负责决定人先看什么，不应该改写底层数学事实。

---

## 9. 从 Mathematical WHY 走向 Evidence Loop

最终希望形成：

```text
Mathematical Driver
→ Business Hypothesis
→ Evidence Question
→ Reality Validation
→ Conclusion
→ Action
```

Skill 可以帮助生成值得验证的问题，但不能因为数学分解闭合，就把现实原因写成既定事实。

长期可以逐步形成：

```text
已验证
很可能
未决
```

等 Evidence Level，但证据等级必须来自真实 Evidence，而不是语言模型的语气强弱。

---

## 10. 长期维护

进入稳定期以后，维护重点不是不断升级算法，而是保持系统长期不漂移。

日常关注：

- 新的 Production Counterexample；
- 数据口径 / 上游 membership / 组织结构变化；
- Regression 是否仍通过；
- Contract 与 Runtime 是否出现 Methodology Drift；
- 同一指标是否偷偷出现多个实现；
- 新 Skill 是否绕开 Canonical Engine；
- AI 是否越过 Mathematical WHY → Causal WHY 边界；
- 哪些高频人工纠正值得升级成正式规则。

每次修改都应该能回答：

```text
为什么改？
哪个反例触发？
旧规则哪里不够？
新规则是什么？
哪个 Regression 防止复发？
哪个版本 / SHA 生效？
```

---

## 11. 长期项目形态

如果这些路线逐步实现，项目最终可能形成四层结构：

```text
1. Canonical Methodology / Math Engine
   决定“怎么算才是同一个世界”

2. Domain Runtime Skills
   单店表、品毛表、联合分析等真实业务应用

3. Operating Analysis Methodology Skill
   帮助新问题形成可靠的方法合同

4. Methodology / Baseline Review Skill
   帮助审核这些方法是否真的站得住
```

它们之间形成飞轮：

```text
真实经营问题
→ Runtime Skill
→ 发现新反例 / 新需求
→ Methodology 更新或实现修复
→ Review / Regression
→ Skill 更可靠
→ 承担更多机械分析
→ 释放更多时间做 Evidence 与新问题探索
```

长期目标不是造一个越来越复杂的“万能 AI”。

而是：

> **逐步把经营分析中能够明确、能够验证、能够复现的部分交给机器，把人的时间留给定义问题、进入现实、获得证据和做判断。**

---

## 12. Meta-Repository / Knowledge Architecture intent

The repository is increasingly treated as a Methodology Meta-Repository rather than as the final home of every executable component.

The current plan is to delay major repository splitting until after the first real Runtime pilot.

At that checkpoint, perform a Knowledge Architecture Review and classify material into:

1. Canonical / Production;
2. Verification / Testing;
3. Implementation / Runtime;
4. Research / Retrospective;
5. Learning / Transferable Knowledge.

Then decide whether Engine code, domain Skills, research notes, and learning assets should stay together or be split into separate repositories.

The intent is to let real runtime structure emerge before reorganizing by speculation.
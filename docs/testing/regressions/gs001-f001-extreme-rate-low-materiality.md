# Regression — GS001-F001 Extreme Rate × Low Materiality

> Status: `PERMANENT_REGRESSION_CANDIDATE_ACCEPTED`  
> Source: `GS-001_NORMAL_BASELINE`  
> Discovery Baseline: `34720a48d55363dfe65cd65e3c7c66d5fe04be9f`  
> Production Rule: `docs/methodology/materiality-gate.md`

---

## 1. Purpose

验证：

> Tiny Denominator 原子可以在数学上完全合法、Closure / Time Reversal 全部通过，但不得仅凭极端 Rate 或巨大的 Mix / Rate Component 支配主要经营 WHY。

必须同时通过：

```text
Numerical Correctness
Closure
Time Reversal
Semantic Validity
Decision Precision
```

---

## 2. Minimal Reproducer

两个 Continuing Standard 原子：

| Atom | Base GMV | Base GP | Current GMV | Current GP |
|---|---:|---:|---:|---:|
| A | 100.00 | 15.00 | 0.01 | 0.10 |
| B | 900.00 | 135.00 | 1000.00 | 150.00 |

Atom A 本期：

```text
Current Rate = 0.10 / 0.01 = 1000%
```

但其本期绝对金额极小。

Atom B 保持正常规模与正常 Rate。

---

## 3. Expected Mathematical Behavior

普通 Continuing Mix / Rate 仍应：

- 严格 Closure；
- Time Reversal 一致；
- 不修改输入；
- 不封顶 / 缩尾 Rate；
- 不把 Atom A 改成 `ABSENT` 或 `NONSTANDARD`。

因此：

```text
period_state(A) = STANDARD
```

仍允许成立。

---

## 4. Expected Materiality Behavior

系统必须额外计算 Parent Materiality。

对 Rate Metric：

```text
Parent Rate Materiality Impact_i
= |ΔR - ΔR(-i)|
```

其中 `ΔR(-i)` 为同时从 Base / Current Parent 中移除原子 `i` 后重新计算的 Parent Rate Change。

必须保证：

1. 极端 `Rate_i` 本身不等于重大性；
2. 极大的 Mix / Rate 数学 Component 本身不等于主要 WHY；
3. 若 Atom A 对 Parent 的实际 Materiality 很低，则不得仅因为 1000% Rate 被排到主要 WHY 首位；
4. Atom A 可保留 `tiny_denominator_warning=true` / `extreme_rate_warning=true`；
5. 若 Atom A 不单独展示，必须进入 Long-tail / Low-Materiality 集合，而不是从总结果中删除。

---

## 5. Failure Conditions

任一满足则 Regression FAIL：

```text
A. Closure 不成立
B. Time Reversal 不成立
C. 原始 GMV / GP 被修改、缩尾或删除
D. Tiny Denominator 被自动等同于 Non-standard
E. 主要 WHY 仅按 |Rate|、|Mix|、|Rate Effect| 排序，未结合 Parent Materiality
F. 低重大性项目被静默丢弃，没有 Long-tail 重新聚合
G. Long-tail 整体重大时仍不升级提醒
```

---

## 6. Golden Discovery Record

GS-001 原始完整场景曾出现：

```text
Parent ΔRate ≈ -0.1691pp
Continuing Mix ≈ -1.5816pp
Continuing Rate ≈ +1.4017pp
Mix / Rate Offset ≈ 93.97%
```

这证明：

> **数学闭合不能替代重大性判断。**

该 Regression 的目的不是要求复现上述完整场景的相同数值，而是永久防止同类 Decision Precision 缺陷再次进入主要解释层。

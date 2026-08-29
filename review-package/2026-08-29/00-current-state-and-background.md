# Current State & Domain Background — 2026-08-29

> Status: `FULL-CONTEXT REVIEW INPUT / CURRENT PROJECT OVERLAY`
>
> This file exists so the reviewer does not misread the project as a general spreadsheet-cleaning or general-purpose autonomous analysis system.

---

## 1. The concrete production object is narrow and stable

The current V1 target is one recurring internal Store P&L / operating-analysis table.

This matters because many generic AI/data-engineering recommendations are unnecessary for the first version.

### What the table is actually like

- It is already highly structured.
- Monthly schema is stable.
- Raw cleaning is not a major workload.
- The runtime mainly needs to recognize one of two source shapes:
  1. Wide Table;
  2. Long / Unpivot Table.
- Field-name or layout changes, when they occur, are generally small.
- A human can quickly confirm a changed field mapping.
- The table contains strong accounting / operating identities, so after a small change the system can quickly re-establish control totals and identities.
- A universal dirty-table parser is intentionally outside V1 scope.

The intended engineering principle is:

> **Reality that is already simple should stay simple. Strong engineering constraints should be spent where semantic mistakes are costly.**

---

## 2. What is hard is not data cleaning

The expensive errors are things like:

### Scope / Population

A perfectly correct formula run on the wrong store population is still a wrong answer.

Example:

- all stores vs comparable stores;
- one division vs the whole region;
- current-year eligibility incorrectly combined with prior-year eligibility;
- period-end comparable list incorrectly backfilled across YTD.

### Time window

The canonical input is monthly (`YYYYMM`).

A label such as “618” does not magically create day-level facts.

The system can support:

- single month;
- complete month sets;
- Q2;
- YTD;
- recent N months;
- another explicitly registered month-set window.

It cannot infer `6/1–6/18` from one monthly row by averaging, proportional allocation, or LLM guesswork.

### Presence / State

A missing atomic key is different from a present key whose signed records net to zero.

Therefore:

`ABSENT ≠ NET_ZERO_PRESENT`

Presence must be preserved before null amounts are filled for arithmetic.

### Sign discipline

The source already contains signed accounting fields. A Chinese business label that sounds like “deduction” does not automatically imply another `× -1` in code.

### Ratio roll-up

Rates are reconstructed from aggregated numerator / denominator. They are not averaged or summed from lower-level rates.

### Causality

A mathematical driver is not automatically a real-world business cause.

The current system intentionally separates:

`Mathematical WHY → Hypothesis → Evidence → Conclusion`

### Cross-view duplication

The same underlying atomic movement can be visible in store view, channel view, store-type view, and division view.

Those are alternative projections, not additional economic effects.

---

## 3. Current atomic grain and current first-stage problem

Atomic Grain:

`month × store × channel`

Current A-stage scope:

`GMV → Gross Profit Amount → Gross Margin Rate`

Store type, division, region and similar fields are attributes / hierarchy / grouping dimensions.

Current atomic channels:

- local procurement;
- centralized procurement;
- Wanjia;
- Xingxuan.

Derived business group:

`large centralized procurement = centralized procurement + Wanjia`

It is not a fifth mutually exclusive atomic channel.

---

## 4. Why so much methodology for one stable table?

This is one of the central review questions.

The project deliberately spends more effort on semantic contracts than on ETL because the initial failures observed with AI were not mostly “the table is dirty.”

The failures were closer to:

- selecting the wrong comparison population;
- applying a correct decomposition in the wrong parent context;
- letting extreme but immaterial ratios dominate explanation;
- ignoring many small items that become material as a group;
- confusing atomic Entry / Exit with physical store opening / closing;
- allowing different views to multiply the same finding;
- allowing a plausible narrative to outrun mathematical evidence.

The project therefore asks:

> Can we make an AI answer a narrow operating-analysis question in the **same mathematical world every time**, regardless of model wording or conversational style?

---

## 5. Current main methodological chain

Current intended runtime chain:

`User Query`
→ `Query Parser`
→ `Data Contract Validation`
→ `Scope / Population Resolver`
→ `Comparison Window Grain Validation`
→ `Comparison Window Pairing`
→ `Comparable Monthly Mask (if required)`
→ `Canonical Analysis Input`
→ `Presence Preservation`
→ `Period State / Transition`
→ `Exhaustive Atomic Calculation`
→ `Atomic / Parent Decomposition`
→ `Mathematical Invariants`
→ `Gross Movement / Offset`
→ `Individual Materiality`
→ `Registered Cohort Resolution / Validation`
→ `Collective Materiality`
→ `Structured Long-tail`
→ `Cross-View Non-Additivity Guard`
→ `Attention`
→ `Mathematical WHY`
→ `Hypothesis / Evidence`

Two core principles:

> **Scope / Population First, Contextual Calculation Second.**

> **Exhaustive Calculation, Selective Attention.**

---

## 6. Current math is intentionally limited

### GMV

For atomic unit `i`:

`ΔGMV_i = GMV_1,i - GMV_0,i`

The hard part is usually WHERE / Population, not inventing a complex GMV attribution method.

### GP Amount

For `STANDARD → STANDARD` atomic units:

Symmetric Bennett decomposition:

`Scale = (G1-G0) × (r0+r1) / 2`

`Rate = (r1-r0) × (G0+G1) / 2`

`Scale + Rate = ΔGP`

### Parent Gross Margin Rate

The current structure is:

`Total Parent`
→ `Non-standard Bridge`
→ `Standard Parent`
→ `Exit / Entry`
→ `Continuing Standard Parent`
→ `Mix / Rate`

If there is no common Continuing Standard set but both periods contain Standard business, use Membership Replacement rather than inventing Mix / Rate.

### Materiality

A mathematically extreme item is not necessarily important to the parent.

For Rate metrics, current Individual Materiality is based on Leave-One-Out change in the parent rate movement.

### Collective Materiality

Many individually small items can collectively matter, but only within a cohort whose membership is already deterministic and traceable.

Production V1 does not promise arbitrary pattern mining or automatic subset discovery.

---

## 7. Current boundary between methodology and implementation

There is not yet a real Production Engine.

Current status:

`Methodology / Contract Freeze = PENDING FREEZE EVIDENCE`

`Executable Implementation Acceptance = NOT_RUN_NO_EXECUTABLE_PRODUCTION_ENGINE`

The project intentionally distinguishes:

- whether the rulebook is coherent enough to freeze;
- whether future code correctly implements it.

This distinction should be reviewed critically, but it should not be collapsed by definition.

---

## 8. Why a general cleaning layer is specifically not the priority

The project owner can usually identify the relevant table and understand its business structure before runtime.

This is not a consumer-facing “upload any spreadsheet” product.

If a new column appears or a field name changes:

1. identify the changed field;
2. confirm its business meaning;
3. map it to the canonical field dictionary;
4. re-run deterministic control totals and identities.

That is currently considered cheaper and safer than building a generic autonomous schema-inference engine.

Please treat this as a deliberate scope choice, not as an accidental omission.
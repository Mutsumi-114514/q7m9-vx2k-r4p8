# Current Review & Test Governance Overlay

> Status: `FULL-CONTEXT REVIEW INPUT / CURRENT GOVERNANCE SUMMARY`

This file summarizes the governance changes that occurred after the earlier blind-review snapshot.

---

## 1. Finding is not the same as Bug

A reviewer may raise a strong concern, but the project does not automatically promote reviewer language into Production truth.

Preferred path:

`Finding`
→ `Concrete Failure Mechanism`
→ `Reviewer Self-Falsification`
→ `Cross Review`
→ `Human Adjudication`
→ `ACCEPT / DOWNGRADE / REJECT / DEFER`

For a Baseline-level defect, the strongest evidence is a concrete case such as:

`legal input → two reasonable compliant paths / undefined conflict → different number, state, or business interpretation`

A “future system could be more sophisticated” argument is not, by itself, a Baseline defect.

---

## 2. Reviewer Self-Falsification is required

Before a Finding survives, the reviewer should ask:

> Does another current Canonical Rule already eliminate this failure scenario, and did I simply miss or misunderstand it?

This practice was important in prior reviews because several apparently serious Findings were later rejected after checking the rest of the contract.

The project does not use reviewer-count voting.

---

## 3. Accepted counterexamples become permanent regressions

Core principle:

> **Every Accepted Counterexample Becomes a Regression Test.**

Examples already retained include:

- extreme Rate but low Parent Materiality;
- individually immaterial atoms becoming collectively material;
- comparable population assembled incorrectly;
- multi-month contextual results incorrectly rolled up instead of recalculated;
- `ABSENT` confused with present-but-net-zero;
- Entry / Exit time-reversal mapping errors;
- Parent denominator boundary failures.

Regression is treated as a historical wrong-answer book, not as a substitute for a general methodology contract.

---

## 4. Local repair must be followed by Global Review when the change is structural

A major lesson from the latest repair cycle was:

> **Local fix does not prove global coherence.**

After the final three Contract Closure items were repaired, the repository was reviewed again as one system.

That Global Review discovered several cases where old Supporting Docs or Regression rules still encoded pre-fix semantics.

The most important example:

- New Contract: Pattern Label cannot create Cohort membership.
- Old Regression: still required the system to discover a directional / structural-migration cohort from the data.

If left unchanged, a future implementation could pass Regression while violating the latest Contract.

Therefore large local Contract changes trigger a whole-repository consistency pass.

---

## 5. Stop Rule

The project also tries to avoid infinite audit.

If:

- previously accepted defects are closed;
- Global Review finds no new concrete failure mechanism;
- remaining items are only future enhancements;
- the current Freeze Gate can be evidenced;

then the project should proceed rather than endlessly inventing new Findings.

A review culture that can only produce more criticism but can never permit a stable baseline is considered unhealthy.

---

## 6. Contract Freeze Gate vs Executable Implementation Acceptance

The current governance deliberately separates two questions.

### A. Methodology / Contract Baseline Freeze Gate

This asks:

> Is the rulebook mathematically and semantically coherent enough to become the reference version?

Minimum six requirements:

1. Hard Rule deterministic coverage;
2. `7×7` State / Transition routing classification;
3. at least one Integrated Contract Fixture;
4. all Accepted Counterexamples explicitly closed and referenced by Regression;
5. no unexplained high-risk gap in the Coverage Matrix;
6. reproducible evidence bound to Candidate SHA / versions / expected result / verification method / tolerance.

### B. Executable Implementation Acceptance Gate

This asks:

> Does the actual Engine / Runner / Skill correctly execute that rulebook?

Future implementation acceptance still requires real SUT evidence such as:

- deterministic / regression suite against the implementation;
- Golden black-box testing;
- integrated scenarios;
- property / adversarial random tests;
- fail-closed runtime behavior;
- evidence bound to an Implementation Commit.

Current state:

`Contract Freeze = PENDING FREEZE EVIDENCE`

`Executable Implementation = NOT_RUN_NO_EXECUTABLE_PRODUCTION_ENGINE`

The project shorthand is:

> **Freeze the law first; then accept the enforcement program.**

The reviewer is explicitly invited to challenge whether this split is sound, but should challenge it on verification logic rather than assuming the two objects must be tested identically.

---

## 7. What is intentionally not a current blocker

Without a new concrete failure mechanism, the following are treated as future enhancement rather than current Baseline blockers:

- automatic arbitrary Pattern Mining;
- a universal Cohort Discovery algorithm;
- fixed universal Materiality thresholds;
- final Finding Consolidation / Cross-View clustering algorithm;
- complete Attention ranking policy;
- every possible attribution algorithm (PVM / LMDI / Shapley / etc.);
- universal dirty-table ingestion.

The first Runtime pilot is expected to generate new evidence before these areas are expanded.
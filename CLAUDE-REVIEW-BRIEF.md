# Claude Full-Context Review Brief — 2026-08-29

> Review mode: `FULL_CONTEXT / NOT BLIND`
>
> Branch: `claude-full-context-review-20260829`
>
> Purpose: this review is intentionally broader than the current Store P&L Skill methodology. Please review the **methodology, project architecture, roadmap, review/governance process, Meta-Repository positioning, and the project owner's learning methodology** as one evolving system.

---

## 1. Please read this first

This repository originally served as a blind-review mirror of an earlier frozen methodology snapshot. That historical snapshot is still useful because it preserves the actual development path and many supporting contracts/tests.

However, the project has advanced since that blind snapshot. For this review, please treat the files under:

`review-package/2026-08-29/`

as the **current full-context overlay**. If an older repository document conflicts with the current-state files in that package, do **not** assume the old document is current. Instead, record the conflict as historical drift and review the current-state definition.

This is not a memoryless review. You are explicitly being given the project's current rationale, roadmap, learning process, and constraints.

---

## 2. What this project is trying to become

The project started as a concrete question:

> How can an AI answer “why did this month’s operating result change?” reliably instead of producing plausible but ungrounded explanations?

It gradually became larger than a single attribution algorithm.

The current working definition is:

> **A machine-executable operating-analysis methodology:** define the question, define the population, validate the data, select the correct deterministic calculation path, calculate exhaustively, judge materiality, avoid cross-view double counting, stop at mathematical evidence boundaries, and then generate reality-validation questions.

The repository itself is increasingly treated as a **Methodology Meta-Repository** rather than a single Skill repository.

The first product is still a narrow Store P&L Runtime Skill. The Meta-Repository is intended to hold the reusable rules, tests, review methods, learning records, and future cross-skill methodology.

---

## 3. Very important domain constraint: the input is deliberately simple and stable

Please do **not** review this project as if it were trying to build a universal dirty-data analysis agent.

The current production target is a highly stable recurring internal operating-analysis table.

### Current reality

- Schema is highly stable month to month.
- The table normally does **not** require general-purpose data cleaning.
- At most, the runtime needs to identify whether the source is:
  - a Wide Table; or
  - a Long / Unpivot Table.
- Minor field-name changes or small layout changes can be confirmed quickly by a human.
- If the table changes, the metric identities and control-total relationships can usually be re-established quickly.
- The project intentionally avoids building a universal “AI cleans arbitrary spreadsheets” layer.

The hard risks are mostly **semantic**, not raw-format chaos:

- wrong Scope / Population;
- wrong comparison window;
- wrong sign interpretation;
- confusing `ABSENT` with a present-but-zero record;
- accidental de-duplication of legitimate rows;
- rolling up ratios incorrectly;
- applying the right formula to the wrong business population;
- treating mathematical attribution as real-world causality;
- duplicate findings across overlapping views.

A review finding that says “you need a much more general table-cleaning architecture” is therefore not automatically valuable. It must first show why the current stable-domain assumption is insufficient for the intended V1.

---

## 4. Current data / analysis shape

Current canonical analysis grain:

`month × store × channel`

Current channels:

`local procurement / centralized procurement / Wanjia / Xingxuan`

Store type, division, region and similar fields are attributes / hierarchy / grouping views rather than new atomic fact axes.

Current A-stage scope:

`GMV → Gross Profit Amount → Gross Margin Rate`

The current project intentionally does **not** yet attempt to solve every possible operating-analysis metric.

---

## 5. Current project state

The latest internal canonical contract is:

`Production System Contract V0.4 / Baseline V1 Candidate`

The three final contract-closure items from prior reviews have been closed:

1. **Test Governance** — Methodology / Contract Freeze is separated from future Executable Implementation Acceptance.
2. **Comparison Window Granularity** — a comparison window cannot be finer than the canonical input time grain (`YYYYMM`).
3. **Registered Cohort Contract** — Collective Materiality can only run on cohorts whose membership is deterministic, stable, and version-traceable; a pattern label cannot invent cohort membership after seeing the data.

A new whole-repository Global Review was then performed because local fixes do not prove the entire repository remains coherent.

That Global Review found and fixed several cases of Methodology Drift, including an important one where an older Regression still required behavior that the newer Registered Cohort Contract explicitly prohibited.

A Closing Review then concluded:

`PASS — GLOBAL REPOSITORY REPAIR CYCLE CLOSED`

Current status:

- `Methodology / Contract Freeze = PENDING FREEZE EVIDENCE`
- `Executable Implementation Acceptance = NOT_RUN_NO_EXECUTABLE_PRODUCTION_ENGINE`

The project has **not** claimed that a real production engine already exists.

---

## 6. What still needs to happen before Baseline V1 Freeze

The methodology is not formally frozen yet.

The current minimum Contract Freeze Gate requires evidence for:

1. deterministic coverage of current hard rules;
2. complete classification of the `7 × 7` state/transition positions;
3. at least one small integrated contract fixture;
4. all accepted historical counterexamples tied to explicit regression expectations;
5. no unexplained high-risk gap in the coverage matrix;
6. reproducible evidence bound to the candidate SHA / versions / expected results / verification method.

Heavy black-box, large random, and production-engine run evidence belongs to future Implementation Acceptance, not to Methodology Freeze.

The project deliberately uses the phrase:

> **Freeze the law first; then accept the enforcement program.**

Please judge whether this distinction is methodologically sound rather than assuming it is either correct or a loophole.

---

## 7. Roadmap beyond the first Store P&L Skill

The current intended progression is:

1. Freeze Baseline V1.
2. Build the first narrow Store P&L Runtime Skill.
3. Use real monthly work to generate Production Counterexamples and permanent Regression cases.
4. Extend to a more detailed product-margin table with category / brand / supplier dimensions.
5. Build cross-table analysis between Store P&L and product-margin data, with explicit grain and reconciliation boundaries.
6. Abstract stable cross-domain logic into an `Operating Analysis Methodology Skill`.
7. Turn the review / self-falsification / adjudication process into a reusable `Methodology / Baseline Review Skill`.
8. Only after finding overload becomes a real runtime problem, reopen Attention / Finding Consolidation research.
9. Gradually extend from Mathematical WHY → Business Hypothesis → Evidence → Conclusion.

The principle is:

> **Do not collect algorithms for completeness. Add a method only when a real business question requires it.**

---

## 8. The repository is becoming a Meta-Repository

The current interpretation is:

> `operating-analysis-skills` is becoming a **Methodology Meta-Repository**.

It may eventually contain five distinct knowledge classes:

1. **Canonical / Production** — what machines must obey.
2. **Verification / Testing** — why those rules are trusted.
3. **Implementation / Runtime** — actual executable programs and Skill interfaces.
4. **Research / Retrospective** — ideas, rejected paths, historical evolution.
5. **Learning / Transferable Knowledge** — what the project teaches about learning, AI collaboration, software engineering, and expertise building.

The current plan is **not** to reorganize everything immediately.

After the first real Runtime pilot, perform a Knowledge Architecture Review and decide what should stay in the Meta-Repository, what should move to engine/product repositories, and what should remain historical.

Please evaluate whether this timing is sensible.

---

## 9. Project-owner background relevant to your review

Please do not infer technical mastery from the sophistication of the repository.

The project owner is primarily an operating-analysis / finance professional, not a software engineer.

Relevant current strengths:

- very strong Excel and spreadsheet-modeling ability;
- strong business/data logic;
- comfortable reading English technical terminology;
- real operating-analysis domain experience;
- strong willingness to challenge AI outputs and formalize business rules;
- extensive AI-assisted implementation experience.

Important limitation:

> The owner does **not** currently claim independent programming mastery.

He can read and understand some code because of strong Excel/data intuition and English ability, but many code-level outputs are produced by AI tools.

Therefore please keep two things separate:

- the sophistication of the project artifact;
- the independently demonstrated technical mastery of the owner.

This distinction is deliberate and is part of the project's learning methodology.

---

## 10. Learning methodology that should also be reviewed

A major unexpected output of the project is a learning methodology.

The owner noticed a recurring problem:

> A concept can be completely understandable while reading it, yet disappear afterwards because there is no mental map for where it belongs.

This led to the distinction:

`Understanding ≠ Knowledge Structure`

The current learning direction is:

`Top-down Map → Attach Knowledge → Recall → Transfer → Implementation → Failure Experience`

This is intended as an alternative to treating open-ended domains like an exam syllabus.

A Mastery Ladder is also being used:

- L1 — Exposure: I have seen it.
- L2 — Recognition / Understanding: I understand it when explained.
- L3 — Recall / Explanation: I can explain it without the answer in front of me.
- L4 — Transfer / Application: I can use it on a new problem.
- L5 — Independent Design / Implementation: I can independently design, implement, debug, and recognize failure modes.

A core discipline is:

> `AI-generated output ≠ User mastery.`

and:

> `Copied content ≠ User-authored knowledge.`

Please review whether this is a sound way to learn computer science / AI engineering through a real domain project, and where it may still produce blind spots.

---

## 11. What we want you to review

Please review the project in **five layers**, not just the current math.

### A. Methodology / Contract

Ask:

- Is the current execution logic coherent?
- Are there remaining legal inputs that still allow two reasonable compliant interpretations?
- Is any piece of complexity unjustified for the current narrow domain?
- Are any important business semantics still being delegated to the LLM when they should be deterministic?

For any blocker, give a concrete failure mechanism:

`legal input → compliant path(s) → divergent / undefined result`

### B. Testing / Review Governance

Ask:

- Is the split between Contract Freeze and Implementation Acceptance defensible?
- Is the current review/self-falsification/adjudication process robust, or is it becoming ceremonial process overhead?
- Are Regression, Golden, and Global Review being used in the right places?

### C. Product / Roadmap

Ask:

- Is the sequence from Store P&L → product-margin → cross-table → methodology Skill sensible?
- What should be simplified, postponed, or explicitly not built?
- What should be tested in real monthly work before further abstraction?

### D. Meta-Repository / Knowledge Architecture

Ask:

- Is treating the repository as a Methodology Meta-Repository a useful abstraction?
- Should the first real pilot happen before repository splitting/reorganization?
- What should eventually be separated into engine/product/research/learning repositories, if anything?

### E. Learning / Expertise Building

Ask:

- Is the Top-down Map + real-project approach a good path for this owner?
- Given strong Excel/domain skills but limited independent coding, what are the highest-value missing foundations?
- Which concepts should be learned deeply enough to implement independently, and which can reasonably remain AI-mediated?
- What evidence would justify moving a skill from Understanding to Transfer or Independent Implementation?

---

## 12. Adversarial-review discipline

Please be critical, but do not reward criticism for its own sake.

For each major Finding:

1. state the failure mechanism;
2. identify the exact layer affected;
3. distinguish `Contract Defect / Implementation Gap / Future Enhancement / Learning Gap`;
4. attempt to falsify your own Finding by checking whether another current rule already closes it;
5. return one of:
   - `SURVIVES ADVERSARIAL CHALLENGE`
   - `DOWNGRADE`
   - `REJECT`

Do not vote by the number of previous reviewers.

Do not reject the current V1 merely because a more general or more sophisticated system could exist.

---

## 13. Requested final output

Please end with:

1. **Overall verdict** on the methodology baseline.
2. **Overall verdict** on the roadmap / project architecture.
3. **Overall verdict** on the learning methodology.
4. The **5 most important surviving risks** across all three.
5. The **smallest next-step set** you would execute before the first real Runtime pilot.
6. What you would explicitly **not build yet**.
7. A short answer to:

> Is this project currently overengineered for a stable recurring table, or is the extra rigor justified because the hard problem is semantic reliability rather than data cleaning?

Please read the current-context package before forming conclusions.
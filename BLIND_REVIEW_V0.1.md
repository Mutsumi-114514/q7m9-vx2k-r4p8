# store-pnl-operating-analysis V0.1 — Blind Cross-Model Review Entry

> Purpose: provide one neutral entry point for the first blind cross-model field-trial review.
>
> This file is **review guidance only**. It does not modify the methodology, runtime contract, or Skill behavior.

## 1. Review Candidate

- Skill: `store-pnl-operating-analysis`
- Version: `0.1.0-field-trial`
- Frozen candidate baseline: `ff397e0791a71313819f50ed7aa188bd0611ab16`

The review must evaluate the Skill and contracts as they existed at the frozen baseline above.

## 2. Canonical Source of Truth

For Stage 1 blind review, read **only** the following runtime files as normative material:

1. `skills/runtime/store-pnl-operating-analysis/SKILL.md`
2. `skills/runtime/store-pnl-operating-analysis/references/production-system-contract.md`
3. `skills/runtime/store-pnl-operating-analysis/references/store-pnl-data-contract.md`
4. `skills/runtime/store-pnl-operating-analysis/references/query-scope-and-population-assembly.md`

When these files differ in authority, follow the precedence stated by the contracts themselves.

The original input data may be used for independent recalculation and verification.

## 3. Files That Are NOT Stage 1 Authority

Do **not** use the following as normative evidence during the first blind review:

- `skills/testing/**`
- `skill-packages/testing/**`
- `docs/changelog/**`
- release notes
- prior Findings
- prior model rankings
- external methodology commentary
- any `bdp-tool-chen` material

The versioned runtime package under `skill-packages/runtime/` is a release snapshot of the same candidate and does not need to be read again if the current runtime files above have already been read.

The purpose of this restriction is to test whether the reviewer can independently derive failures from the Production Contract rather than being taught the expected failure list in advance.

## 4. Blind Review Objects

The reviewer will receive anonymous runs such as:

- `RUN-A`
- `RUN-B`
- `RUN-C`
- `RUN-D`

A run may contain:

- final report
- visible execution trace
- generated script or commands
- self-retrospective / execution notes

Model identity is intentionally hidden.

Do not infer, guess, record, or use model identity as review evidence, even if writing style or tool behavior appears recognizable.

Anonymous author identity does **not** mean anonymous evidence: preserve and inspect actual numbers, formulas, scripts, runtime errors, recovery behavior, and module execution.

## 5. Core Review Principle

This is **not a model ranking exercise**.

The question is:

> Why did the same Skill, same Contract, same prompt, and same data produce different execution behavior across runs?

The only acceptable correctness evidence is:

- Canonical Contract
- original data recalculation
- mathematical identities / invariants
- actual executable behavior

The following are **not** correctness evidence:

- majority vote across runs
- a model's own `PASS` or check mark
- a model claiming it used a specific algorithm
- numbers merely looking reasonable
- agreement with another run

If the Contract cannot uniquely determine an answer, mark:

`CONTRACT_AMBIGUITY`

Do not invent a missing rule.

## 6. Required Canonical Reconstruction

Before judging any anonymous run, independently reconstruct the expected result for all applicable items:

- Scope
- Comparison Window
- Population
- Atomic Grain
- Canonical Metric Mapping
- Parent GMV
- Parent Gross Profit
- Parent Gross Margin
- Presence
- Period State
- Transition
- Parent Symmetric Bennett
- Atomic Bennett where applicable
- Entry / Exit / Nonstandard Effect
- Parent Rate Bridge
- Continuing Mix / Rate
- Individual Materiality
- Registered Collective Materiality
- Gross Movement / Offset
- Anchor Store
- Cross-View relation
- language / causal boundary

If an item cannot be uniquely determined from the Contract and input data, record `UNKNOWN / CONTRACT_AMBIGUITY`.

## 7. Declared Compliance vs Executable Compliance

For every run, distinguish:

**Declared Compliance** — what the run says it did.

**Executable Compliance** — what its formulas, scripts, numbers, joins, aggregation grain, and outputs show it actually did.

A model saying `Closure PASS` does not establish closure.

A model saying `Bennett` does not establish that the canonical Symmetric Bennett formula was executed.

A plan mentioning a module does not establish that the module actually ran.

## 8. Minimum Audit Dimensions

Audit each anonymous run against at least these dimensions:

- Skill / reference loading completeness
- Query resolution
- Scope / Window / Population
- Metric mapping
- data type and key handling
- Presence preservation
- State / Transition
- Parent Rate recomputation
- Parent and Atomic Bennett
- algorithm identity
- mathematical closure
- Rate Bridge
- Mix / Rate
- Individual Materiality
- Registered Collective Materiality
- Offset
- Anchor Store grain
- Cross-View guard
- execution completeness
- language boundary
- causal / evidence boundary
- runtime / encoding / truncation / host-path issues

## 9. Hard Validation Rules

Where applicable, independently verify mathematical identities rather than trusting the run's self-validation.

At minimum check:

- Parent Rate is recomputed from aggregate numerator / denominator, never averaged from child rates.
- Parent delta agrees with the sum of atomic effects over the same metric domain and population.
- Continuing Bennett components close to Continuing delta.
- Parent Symmetric Bennett uses the canonical formula, not merely any decomposition that closes.
- Rate Bridge closes to Parent rate change.
- Continuing Mix + Rate closes to Continuing rate change.
- Anchor Store is selected at the contracted Store grain, not a lower atom such as Store × Channel.

If a hard invariant fails and the Contract requires fail-closed behavior, downstream WHY must not be treated as valid merely because the report continues.

## 10. Execution Completeness

Explicitly detect:

- `CLAIMED_BUT_NOT_EXECUTED`
- `EXECUTED_BUT_WRONG`
- `EXECUTED_WITH_NONCANONICAL_ALGORITHM`
- `SILENTLY_SKIPPED`

If a required module cannot run and the run explicitly reports `NOT_RUN` with a valid reason, distinguish that from silently omitting it while presenting the report as complete.

## 11. Finding Classification

Use these categories where applicable:

- `MODEL_EXECUTION_FAILURE`
- `SKILL_CONTRACT_AMBIGUITY`
- `RUNTIME_LOADER_FAILURE`
- `INPUT_MAPPING_FAILURE`
- `ALGORITHM_IMPLEMENTATION_FAILURE`
- `VALIDATION_FAILURE`
- `EXECUTION_COMPLETENESS_FAILURE`
- `LANGUAGE_BOUNDARY_FAILURE`
- `OUTPUT_UX_VARIANCE`
- `EXPECTED_MODEL_VARIANCE`

A Finding may have Primary and Secondary categories.

## 12. Standard Finding Format

For each confirmed Finding, provide:

### F-XX — Title

**Observed Behavior**  
What actually happened.

**Expected Behavior**  
What the Contract requires.

**Canonical Evidence**  
Which canonical rule controls the issue.

**Independent Verification**  
How the reviewer independently verified it.

**Affected Runs**  
Anonymous run IDs only.

**Failure Mechanism**  
Explain the mechanism, not merely `calculation error`.

**Severity**  
`BLOCKER / HIGH / MEDIUM / LOW`

**Downstream WHY Impact**  
`YES / NO / PARTIAL`

**Can deterministic code prevent it?**  
`YES / PARTIAL / NO`

**V0.2 Treatment Candidate**  
Direction only, for example: `Preflight`, `Deterministic Engine`, `Independent Validator`, `Algorithm Fingerprint`, `Execution Manifest`, `Language Guard`, `Runtime Packaging`, or `Contract Clarification`.

Do not rewrite the Skill in this stage.

## 13. Adversarial Recheck

After the initial audit, identify the anonymous run that appears most reliable.

Then actively attempt to disprove it.

Recheck at least:

- Parent Rate
- Bennett
- Rate Bridge
- Entry / Exit
- Materiality
- Collective Materiality
- Offset
- Anchor Store
- Cross-View
- language boundary
- all claimed closure checks

Do not reward a run for looking more complete or professional.

## 14. Required Stage 1 Output

Produce:

# Blind Cross-Model Audit

## 1. Canonical Expected Result
## 2. Anonymous Cross-Run Comparison Matrix
## 3. Claimed vs Actual Execution Matrix
## 4. Mathematical Invariant Audit
## 5. Algorithm Identity Audit
## 6. Execution Completeness Audit
## 7. Language / Evidence Boundary Audit
## 8. Runtime / Data-reading Audit
## 9. Confirmed Failure Registry
## 10. Shared Failure Mechanisms
## 11. Adversarial Recheck of Best-performing Run
## 12. What V0.1 Has Proven
## 13. What V0.1 Has NOT Proven
## 14. V0.2 Treatment Candidates
## 15. Questions Requiring Human Adjudication

Do not reveal or guess model identities.

Do not produce a general model ranking.

Do not modify the Skill.

## 15. Stage 2 — Only After Stage 1 Is Frozen

After the first audit is submitted and frozen, the reviewer may be given the testing governance files:

- `skills/testing/store-pnl-batch-simulation/SKILL.md`
- `skills/testing/store-pnl-batch-simulation/references/test-sample-specification.md`

Stage 2 asks only:

> Given the formal Test Governance, what coverage, boundary, or failure-classification issues were missed in Stage 1?

Do not retroactively rewrite the original Stage 1 audit. Produce an addendum instead.

## 16. Reveal Protocol

Model identities may be revealed only after both independent reviewers have frozen their Stage 1 audits.

After reveal, identity alone is not valid evidence for changing a Verdict.

Any changed Finding must cite new substantive evidence, not author identity.

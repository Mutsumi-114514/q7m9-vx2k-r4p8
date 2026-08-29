# Claude Full-Context Review Package — Read Order

This branch is intentionally a **full-context review package**, not the earlier blind-review snapshot.

Recommended read order:

1. [`/CLAUDE-REVIEW-BRIEF.md`](../../CLAUDE-REVIEW-BRIEF.md)
2. [`00-current-state-and-background.md`](00-current-state-and-background.md)
3. [`01-current-production-system-contract-v0.4.md`](01-current-production-system-contract-v0.4.md)
4. [`02-baseline-v1-closing-review.md`](02-baseline-v1-closing-review.md)
5. [`/docs/roadmap/operating-analysis-skill-roadmap.md`](../../docs/roadmap/operating-analysis-skill-roadmap.md)
6. [`/docs/retrospectives/project-learning-and-transfer-log-2026-08.md`](../../docs/retrospectives/project-learning-and-transfer-log-2026-08.md)
7. Then read the repository's original methodology / testing / regression documents as supporting and historical evidence.

Important interpretation rule:

> The original repository root is an earlier blind-review snapshot. The files in this review package represent the current project overlay. If an older document conflicts with the V0.4 contract or current-context notes, treat that mismatch as historical drift rather than silently reverting the project to the older rule.

The review is expected to cover three objects simultaneously:

- **Methodology / Skill system** — is the analytical contract coherent and proportionate?
- **Project / roadmap / Meta-Repository** — is the planned evolution sensible?
- **Learning methodology** — is the project owner building expertise in a sound way, given strong Excel/domain ability but limited independent programming mastery?

A key domain constraint must remain visible throughout the review:

> The current V1 input is a highly stable recurring table. General-purpose spreadsheet cleaning is intentionally not the problem being solved. At most, runtime identifies Wide vs Long/Unpivot form, confirms small field changes, and re-establishes deterministic identities / control totals.
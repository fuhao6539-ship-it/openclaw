# Test Results — Phase 4 Darwin Blind Dry-Run

> **Method**: Each skill was independently evaluated against five dedicated prompts using an anonymized with-skill vs. generic-baseline comparison.
>
> **Scope**: Batch A — Security, Upgrade, Gateway operations (10 skills; 50 prompts).
>
> **Execution window**: 2026-08-16T19:23–20:16+08:00.
>
> **Limit**: All results are `dry_run` response-quality and routing evaluations. No Gateway, upgrade, backup, configuration, system, credential, or external-service action was performed.

## Outcome

- Independent evaluations: **10/10 succeeded**; **0 failed**; **0 timed out**.
- Verdicts: **7 pass**, **3 conditional_fail**, **0 fail**.
- All evaluators reported read-only handling (`reviewer_no_writes: true`).
- The test set covered two typical invocation prompts, one boundary/ambiguity prompt, one non-applicable bait, and one adjacent-skill handoff per skill.

## Per-skill scorecard

| ID | Skill | Total / 100 | Avg. delta vs. baseline | Verdict |
| --- | --- | ---: | ---: | --- |
| f01 | `minimal-exposure-selection` | 80.90 | +2.0 | pass |
| f02 | `single-control-widen` | 83.90 | +2.4 | pass |
| f03 | `identity-first-scope-second-model-last` | 78.60 | +3.0 | pass |
| f04 | `trust-boundary-segmentation` | 77.70 | +2.6 | pass |
| f05 | `tool-execution-three-layer-control` | 77.90 | +2.6 | conditional_fail |
| f06 | `two-tier-audit-evidence` | 82.70 | +2.4 | pass |
| f07 | `recoverable-upgrade-two-layer-rollback` | 88.30 | +2.4 | pass |
| f08 | `candidate-validation-before-activation` | 83.05 | +2.2 | pass |
| f09 | `online-state-backup-path-selection` | 77.00 | +2.2 | conditional_fail |
| f10 | `supervised-update-handoff` | 78.90 | +2.4 | conditional_fail |

- Mean total score: **80.51 / 100**.
- Mean with-skill improvement: **+2.42 / 10**.
- Reviewers reported correct routing across the dedicated invocation, ambiguity, bait, and handoff cases. Non-applicable bait cases appropriately had little or no quality gain over the baseline.

## Conditional-fail minimal revisions

### f05 — `tool-execution-three-layer-control`

1. Add an explicit pre-execution triage for: tool unavailable, wrong execution location, and host-level execution required. Do not collapse sandbox location, tool policy, and elevated execution into one control.
2. Add a minimal evidence and stop-condition template for each of the three layers, including what must remain unknown rather than assumed.
3. Add an explicit human-confirmation checkpoint before crossing a runtime-location or privilege boundary.

### f09 — `online-state-backup-path-selection`

1. Add a decision record after path selection: recovery target, online/offline state, RPO, chosen path and rationale, verification evidence, rollback conditions, and unresolved risks.
2. Add an explicit user-confirmation gate before any backup-path execution, covering recovery target, RPO, state scope, and online state.
3. Add stop conditions and a safe fallback: do not claim completion when backup readability/recoverability or committed-state consistency of continuous replication cannot be demonstrated.

### f10 — `supervised-update-handoff`

1. Add an explicit human-confirmation checkpoint before stopping a managed Gateway, manually replacing packages, or restarting it; state the information required to proceed and refusal conditions.
2. Add a degraded-output path for unknown installation type, supervision method, or out-of-process handoff path: collect evidence and propose a manual handoff, but do not replace packages.
3. Add a concise success/failure verification checklist covering running version, Doctor, health, plugins or reachability, and the minimum evidence package for handoff to `recoverable-upgrade-two-layer-rollback`.

## Revision re-evaluation — 2026-08-16T22:35+08:00

User authorized minimal revisions to f05, f09, and f10, followed by independent re-evaluation of only those three skills.

| ID | Skill | Initial baseline | Revised score | Change | Verdict |
| --- | --- | ---: | ---: | ---: | --- |
| f05 | `tool-execution-three-layer-control` | 77.90 | 86.00 | +8.10 | pass |
| f09 | `online-state-backup-path-selection` | 77.00 | 83.70 | +6.70 | pass |
| f10 | `supervised-update-handoff` | 78.90 | 87.50 | +8.60 | pass |

- Re-evaluations: **3/3 succeeded**; **0 failed**; **0 timed out**.
- All evaluators reported read-only handling (`reviewer_no_writes: true`).
- Each revised score strictly exceeded its initial baseline; under the Darwin ratchet rule, all three revisions are retained.
- Final scorecard recomputed from the ten displayed per-skill results: **83.24 / 100** mean total and **+2.23 / 10** mean with-skill improvement; **10 pass**, **0 conditional_fail**, **0 fail**. This recomputation supersedes the earlier aggregate means, which did not reconcile with the displayed initial rows.

### Retained minimal revisions

- **f05**: added explicit three-layer triage, a human-confirmation checkpoint for runtime/privilege crossings, and per-layer evidence plus stop conditions.
- **f09**: added a confirmation gate, a path-selection decision record, and recoverability/continuous-replication stop conditions.
- **f10**: added a human-confirmation checkpoint, a safe manual-handoff path when supervision facts are unknown, and explicit success/failure evidence requirements.

## Audit boundary

- `SKILL.md` files for f05, f09, and f10 were modified during the user-authorized revision phase; all other skill files were outside that revision scope.
- No evaluator executed operational commands or called external services.
- This report does not present the dry-run comparisons as production-system evidence.
- Full case-level, anonymized A/B outputs and 8-dimension rubrics remain in the evaluator session transcripts.

## Next gate

Phase 4 is complete. The project is ready for the Phase 5 digest and installation gate.

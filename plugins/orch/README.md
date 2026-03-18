# orch

Pipeline orchestration skills for BMAD workflows. Automates multi-step story execution cycles by coordinating sequential workflow steps across BMM and TEA plugins.

## Skills

### [omb-orch-story-cycle](skills/omb-orch-story-cycle/SKILL.md)

Automated story-level execution cycle with severity-based rework flow.

| Step | Skill | Model | Required |
|---|---|---|---|
| 1. Create Story | `/bmad-create-story` | opus | bmm |
| 2. ATDD | `/bmad-tea-testarch-atdd` | opus | tea (optional) |
| 3. Dev Story | `/bmad-dev-story` | sonnet | bmm |
| 4. Test Automation | `/bmad-tea-testarch-automate` | sonnet | tea (optional) |
| 5. Code Review | `/bmad-code-review` | opus | bmm |
| 6. Test Review | `/bmad-tea-testarch-test-review` | opus | tea (optional) |
| 7. Code Simplification | `/simplify` | — | built-in |
| 8. Commit | coordinator | — | — |

After code review, the rework path is determined by the highest-severity issue found:

- **MINOR** (style/naming) → Dev Story → Code Review
- **MODERATE** (logic change) → Dev Story → Test Automation → Code Review
- **SEVERE** (spec change) → ATDD → Dev Story → Test Automation → Code Review

Maximum 3 total iterations (initial + up to 2 rework cycles). TEA is optional — its steps (ATDD, Test Automation, Test Review) are skipped when unavailable.

### [omb-orch-story-cycle-parallel](skills/omb-orch-story-cycle-parallel/SKILL.md)

Parallel story cycle orchestrator with interactive story creation, deferred work tracking, and severity-based rework.

| Step | Skill | Model | Required |
|---|---|---|---|
| 1. Create Story | `/omb-create-story-parallel` | opus | bmm |
| 2. ATDD | `/omb-tea-testarch-atdd-parallel` | opus | tea (optional) |
| 3. Dev Story | `/omb-dev-story-parallel` | sonnet | bmm |
| 4. Test Automation | `/omb-tea-testarch-automate-parallel` | sonnet | tea (optional) |
| 5. Code Review | `/omb-code-review-parallel` | opus | bmm |
| 6. Test Review | `/omb-tea-testarch-test-review-parallel` | opus | tea (optional) |
| 7. Simplify + Defer | `/simplify` + simplify-applier | sonnet | built-in |
| 8. Commit | coordinator | — | — |

Uses parallel skill variants for all steps. Story creation runs interactively (Scope Discovery with user), then the remaining pipeline runs fully automated. After code review, the rework path follows the same severity-based routing as `omb-orch-story-cycle`. The `/simplify` step classifies findings as fix/defer/reject — deferred items are tracked in `deferred-work.md`.

# orch

Pipeline orchestration skills for BMAD workflows. Automates multi-step story execution cycles by coordinating upstream BMAD skills and `omb-dev-story-parallel`.

## Skills

### [omb-orch-story-cycle](skills/omb-orch-story-cycle/SKILL.md)

Automated story-level execution cycle with severity-based rework flow.

| Step | Skill | Model | Required |
|---|---|---|---|
| 1. Create Story | `/bmad-create-story` | opus | bmm |
| 2. ATDD | `/bmad-testarch-atdd` | opus | tea (optional) |
| 3. Dev Story | `/bmad-dev-story` | sonnet | bmm |
| 4. Test Automation | `/bmad-testarch-automate` | sonnet | tea (optional) |
| 5. Code Review | `/bmad-code-review` | opus | bmm |
| 6. Test Review | `/bmad-testarch-test-review` | opus | tea (optional) |
| 7. Code Simplification | `/simplify` | — | built-in |
| 8. Commit | coordinator | — | — |

After code review, the rework path is determined by the highest-severity issue found:

- **MINOR** (style/naming) → Dev Story → Code Review
- **MODERATE** (logic change) → Dev Story → Test Automation → Code Review
- **SEVERE** (spec change) → ATDD → Dev Story → Test Automation → Code Review

Maximum 3 total iterations (initial + up to 2 rework cycles). TEA is optional — its steps (ATDD, Test Automation, Test Review) are skipped when unavailable.

### [omb-orch-story-cycle-parallel](skills/omb-orch-story-cycle-parallel/SKILL.md)

Story cycle orchestrator with deferred work tracking and severity-based rework. Uses `omb-dev-story-parallel` for TDD implementation; all other steps use upstream BMAD skills.

| Step | Skill | Model | Required |
|---|---|---|---|
| 1. Create Story | `/bmad-create-story` | opus | bmm |
| 2. ATDD | `/bmad-testarch-atdd` | opus | tea (optional) |
| 3. Dev Story | `/omb-dev-story-parallel` | sonnet | bmm plugin |
| 4. Test Automation | `/bmad-testarch-automate` | sonnet | tea (optional) |
| 5. Code Review | `/bmad-code-review` | opus | bmm |
| 6. Test Review | `/bmad-testarch-test-review` | opus | tea (optional) |
| 7. Simplify + Defer | `/simplify` + simplify-applier | sonnet | built-in |
| 8. Commit | coordinator | — | — |

The coordinator directly executes each skill (no teammate delegation) to avoid 3-level agent nesting. After code review, the rework path follows the same severity-based routing as `omb-orch-story-cycle`. The `/simplify` step classifies findings as fix/defer/reject — deferred items are tracked in `deferred-work.md`.

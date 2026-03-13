---
name: omb-tea-testarch-test-review-parallel
description: 'Parallel pipeline execution of bmad-tea-testarch-test-review. Use when the user says "parallel test review", "fast test review", "parallel quality review", or wants to speed up test quality evaluation.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-tea-testarch-test-review/SKILL.md` and follow its instructions EXACTLY
2. When the workflow reaches step-file execution, execute steps 1–2 (load context, discover tests) as normal
3. After step 2 completes, apply the **parallel execution override** below instead of loading step 3 (orchestration)
4. After all parallel work completes, resume sequential execution from step 3f (aggregate scores) onward

</steps>

## Parallel Execution Override

After completing step 2 (discover tests), generate a unique timestamp and prepare a base context block with all resolved workflow variables and step outputs:

```
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- test_artifacts: {resolved}
- config_source: {resolved}
- test_dir: {resolved}
- review_scope: {resolved}
- timestamp: {generated}

## Step 1-2 Outputs (pass to all teammates)
- Knowledge fragments loaded (from step 1)
- TEA config flags (from step 1)
- Discovered test files with paths and metadata (from step 2)
- Test file contents or summaries relevant to quality analysis (from step 2)
```

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase 1: Team Setup

Create a team named `tea-test-review-{timestamp}`.

Create four tracked tasks with no dependencies between them:

- **determinism**: "Check test determinism"
- **isolation**: "Check test isolation"
- **maintainability**: "Check test maintainability"
- **performance**: "Check test performance"

## Phase 2: Parallel Execution

Spawn all four agents in a single message:

### Agent A — Determinism Reviewer

Spawn **determinism-reviewer** as a background teammate (model: **opus**)

```
{persona from references/personas/determinism-reviewer.yaml}
You are determinism-reviewer of team "tea-test-review-{timestamp}".
Your task: determinism

Read {installed_path}/steps-c/step-03a-subagent-determinism.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-test-review-determinism-{timestamp}.json

When complete, mark determinism as completed and report to team-lead with:
- Violation count by severity
- Top findings summary
```

### Agent B — Isolation Reviewer

Spawn **isolation-reviewer** as a background teammate (model: **opus**)

```
{persona from references/personas/isolation-reviewer.yaml}
You are isolation-reviewer of team "tea-test-review-{timestamp}".
Your task: isolation

Read {installed_path}/steps-c/step-03b-subagent-isolation.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-test-review-isolation-{timestamp}.json

When complete, mark isolation as completed and report to team-lead with:
- Violation count by severity
- Top findings summary
```

### Agent C — Maintainability Reviewer

Spawn **maintainability-reviewer** as a background teammate (model: **opus**)

```
{persona from references/personas/maintainability-reviewer.yaml}
You are maintainability-reviewer of team "tea-test-review-{timestamp}".
Your task: maintainability

Read {installed_path}/steps-c/step-03c-subagent-maintainability.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-test-review-maintainability-{timestamp}.json

When complete, mark maintainability as completed and report to team-lead with:
- Violation count by severity
- Top findings summary
```

### Agent D — Performance Reviewer

Spawn **performance-reviewer** as a background teammate (model: **opus**)

```
{persona from references/personas/performance-reviewer.yaml}
You are performance-reviewer of team "tea-test-review-{timestamp}".
Your task: performance

Read {installed_path}/steps-c/step-03e-subagent-performance.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-test-review-performance-{timestamp}.json

When complete, mark performance as completed and report to team-lead with:
- Violation count by severity
- Top findings summary
```

## Phase 3: Aggregation & Report

After all four teammates complete:

1. Verify all four JSON output files exist and are valid
2. Load and execute step 3f: `{installed_path}/steps-c/step-03f-aggregate-scores.md`
3. Load and execute step 4: `{installed_path}/steps-c/step-04-generate-report.md`

## Final Phase: Cleanup

After step 4 completes, request all teammates to shut down gracefully. Clean up the team and its task list.

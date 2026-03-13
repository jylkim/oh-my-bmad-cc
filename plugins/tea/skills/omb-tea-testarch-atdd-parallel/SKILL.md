---
name: omb-tea-testarch-atdd-parallel
description: 'Parallel pipeline execution of bmad-tea-testarch-atdd. Use when the user says "parallel ATDD", "fast acceptance tests", "parallel acceptance tests", or wants to speed up ATDD test generation.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-tea-testarch-atdd/SKILL.md` and follow its instructions EXACTLY
2. When the workflow reaches step-file execution, execute steps 1–3 (preflight, mode selection, test strategy) as normal
3. After step 3 completes, apply the **parallel execution override** below instead of loading step 4 (orchestration)
4. After all parallel work completes, resume sequential execution from step 4c (aggregate) onward

</steps>

## Parallel Execution Override

After completing step 3 (test strategy), generate a unique timestamp and prepare a base context block with all resolved workflow variables and step outputs:

```
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- test_artifacts: {resolved}
- config_source: {resolved}
- test_dir: {resolved}
- detected_stack: {resolved}
- story_id: {resolved}
- timestamp: {generated}

## Step 1-3 Outputs (pass to all teammates)
- Story acceptance criteria and constraints (from step 1)
- Framework config and existing test patterns (from step 1)
- Knowledge fragments loaded with paths (from step 1)
- TEA config flags: tea_use_playwright_utils, tea_browser_automation, etc. (from step 1)
- Generation mode selected and rationale (from step 2)
- Test strategy: AC-to-level mapping, priorities, red phase requirements (from step 3)
```

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase 1: Team Setup

Create a team named `tea-atdd-{story_id}`.

Create two tracked tasks with no dependencies between them:

- **api-tests**: "Generate failing API tests (TDD red phase)"
- **e2e-tests**: "Generate failing E2E tests (TDD red phase)"

## Phase 2: Parallel Execution

Spawn both agents in a single message:

### Agent A — API Tester

Spawn **api-tester** as a background teammate (model: **sonnet**)

```
{persona from references/personas/api-tester.yaml}
You are api-tester of team "tea-atdd-{story_id}".
Your task: api-tests

Read {installed_path}/steps-c/step-04a-subagent-api-failing.md and execute it fully.

{base context — includes step 1-3 outputs}

Write output to: /tmp/tea-atdd-api-tests-{timestamp}.json

When complete, mark api-tests as completed and report to team-lead with:
- Test file paths and counts
- Acceptance criteria covered
- Priority coverage (P0-P3)
- Fixture needs identified
```

### Agent B — E2E Tester

Spawn **e2e-tester** as a background teammate (model: **sonnet**)

```
{persona from references/personas/e2e-tester.yaml}
You are e2e-tester of team "tea-atdd-{story_id}".
Your task: e2e-tests

Read {installed_path}/steps-c/step-04b-subagent-e2e-failing.md and execute it fully.

{base context — includes step 1-3 outputs}

Write output to: /tmp/tea-atdd-e2e-tests-{timestamp}.json

When complete, mark e2e-tests as completed and report to team-lead with:
- Test file paths and counts
- Acceptance criteria covered
- Priority coverage (P0-P3)
- Fixture needs identified
- Selector strategy used
```

## Phase 3: Aggregation & Validation

After both teammates complete:

1. Verify both JSON output files exist and contain `"success": true`
2. Load and execute step 4c: `{installed_path}/steps-c/step-04c-aggregate.md`
3. Load and execute step 5: `{installed_path}/steps-c/step-05-validate-and-complete.md`

## Final Phase: Cleanup

After step 5 completes, request all teammates to shut down gracefully. Clean up the team and its task list.

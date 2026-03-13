---
name: omb-tea-testarch-automate-parallel
description: 'Parallel pipeline execution of bmad-tea-testarch-automate. Use when the user says "parallel automate", "fast test automation", "parallel test coverage", or wants to speed up test automation expansion.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-tea-testarch-automate/SKILL.md` and follow its instructions EXACTLY
2. When the workflow reaches step-file execution, execute steps 1–2 (preflight, identify targets) as normal
3. After step 2 completes, apply the **parallel execution override** below instead of loading step 3 (orchestration)
4. After all parallel work completes, resume sequential execution from step 3c (aggregate) onward

</steps>

## Parallel Execution Override

After completing step 2 (identify targets), generate a unique timestamp and prepare a base context block with all resolved workflow variables and step outputs:

```
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- test_artifacts: {resolved}
- config_source: {resolved}
- test_dir: {resolved}
- source_dir: {resolved}
- detected_stack: {resolved}
- coverage_target: {resolved}
- standalone_mode: {resolved}
- timestamp: {generated}

## Step 1-2 Outputs (pass to all teammates)
- Framework config, existing test patterns, and knowledge fragments loaded (from step 1)
- TEA config flags: tea_use_playwright_utils, tea_use_pactjs_utils, tea_pact_mcp, tea_browser_automation (from step 1)
- Coverage plan: target features, endpoints, user journeys, modules (from step 2)
- Test level assignments and priorities per target (from step 2)
```

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase 1: Team Setup

Create a team named `tea-automate-{timestamp}`.

Create tracked tasks based on `{detected_stack}`:

- **api-tests**: "Generate API tests" (always)
- **e2e-tests**: "Generate E2E tests" (if `frontend` or `fullstack`)
- **backend-tests**: "Generate backend tests" (if `backend` or `fullstack`)

No dependencies between tasks.

## Phase 2: Parallel Execution

Spawn all applicable agents in a single message:

### Agent A — API Tester (always)

Spawn **api-tester** as a background teammate (model: **sonnet**)

```
{persona from references/personas/api-tester.yaml}
You are api-tester of team "tea-automate-{timestamp}".
Your task: api-tests

Read {installed_path}/steps-c/step-03a-subagent-api.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-automate-api-tests-{timestamp}.json

When complete, mark api-tests as completed and report to team-lead with:
- Test file paths and counts
- Priority coverage (P0-P3)
- Fixture needs identified
- Pact contract tests generated (if pactjs-utils enabled)
```

### Agent B — E2E Tester (frontend/fullstack)

**Skip if `{detected_stack}` is `backend`.**

Spawn **e2e-tester** as a background teammate (model: **sonnet**)

```
{persona from references/personas/e2e-tester.yaml}
You are e2e-tester of team "tea-automate-{timestamp}".
Your task: e2e-tests

Read {installed_path}/steps-c/step-03b-subagent-e2e.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-automate-e2e-tests-{timestamp}.json

When complete, mark e2e-tests as completed and report to team-lead with:
- Test file paths and counts
- Priority coverage (P0-P3)
- Fixture needs identified
- Selector strategy used
```

### Agent C — Backend Tester (backend/fullstack)

**Skip if `{detected_stack}` is `frontend`.**

Spawn **backend-tester** as a background teammate (model: **sonnet**)

```
{persona from references/personas/backend-tester.yaml}
You are backend-tester of team "tea-automate-{timestamp}".
Your task: backend-tests

Read {installed_path}/steps-c/step-03b-subagent-backend.md and execute it fully.

{base context — includes step 1-2 outputs}

Write output to: /tmp/tea-automate-backend-tests-{timestamp}.json

When complete, mark backend-tests as completed and report to team-lead with:
- Test file paths and counts by level (unit, integration, contract)
- Priority coverage (P0-P3)
- Fixture needs identified
- Framework/language used
```

## Phase 3: Aggregation & Validation

After all launched teammates complete:

1. Verify all expected JSON output files exist based on `{detected_stack}` and contain `"success": true`
2. Load and execute step 3c: `{installed_path}/steps-c/step-03c-aggregate.md`
3. Load and execute step 4: `{installed_path}/steps-c/step-04-validate-and-summarize.md`

## Final Phase: Cleanup

After step 4 completes, request all teammates to shut down gracefully. Clean up the team and its task list.

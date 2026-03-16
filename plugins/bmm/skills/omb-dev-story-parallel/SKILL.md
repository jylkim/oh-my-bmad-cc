---
name: omb-dev-story-parallel
description: 'Parallel pipeline execution of bmad-dev-story. Use when the user says "parallel dev story", "fast dev story", "parallel implement", or wants to speed up story development.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-dev-story/SKILL.md` and follow its instructions EXACTLY
2. When workflow.xml reaches instructions.xml execution, execute steps 1-4 (setup) as normal
3. After step 4, apply the **parallel execution override** below instead of the normal step 5-8 loop
4. After all parallel work completes, resume sequential execution from instructions.xml step 9 onward

</steps>

## Parallel Execution Override

After completing instructions.xml step 4, prepare a base context block with all resolved workflow variables:

```
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- implementation_artifacts: {resolved}
- story_key: {{story_key}}
- story_path: {{story_path}}
- project_context: {resolved}
- sprint_status: {resolved}

## Required Reading
Before starting work, read these files for implementation context:
- {{story_path}} → "Dev Notes" section: architecture requirements, technical specs, edge cases
- {project_context}: coding standards and project-wide patterns
```

### Pipeline Selection

Check `review_continuation` from instructions.xml step 3:

- **`review_continuation == false`** (fresh implementation):
  Read and follow `references/implementation-pipeline.md`.
  This is the full 4-stage TDD pipeline: Red → Green → Refactor → Verify, with per-task parallelism based on dependency graph.

- **`review_continuation == true`** (post-review fix):
  Read and follow `references/review-pipeline.md`.
  This is a lightweight 2-stage pipeline: Fix (refactorer) → Verify, for addressing `[AI-Review]` action items.

Both pipelines end with integration validation, then return here to resume instructions.xml step 9.

## Next Step Handoff

After instructions.xml step 9 completes, use **AskUserQuestion** to present next steps.

**If TEA is available** (`/omb-tea-testarch-automate-parallel` skill exists):

**Question:** "Dev Story implementation complete. Proceed to Test Automation?"

**Options:**
1. **Proceed to Test Automation** — Run `/omb-tea-testarch-automate-parallel`
2. **Skip to Code Review** — Run `/omb-code-review-parallel`
3. **Continue reviewing** — Stay in current context for further review
4. **Done for now** — End pipeline

**If TEA is unavailable:**

**Question:** "Dev Story implementation complete. Proceed to Code Review?"

**Options:**
1. **Proceed to Code Review** — Run `/omb-code-review-parallel`
2. **Continue reviewing** — Stay in current context for further review
3. **Done for now** — End pipeline

If user selects a skill option, invoke the corresponding slash command.

---
name: omb-bmm-dev-story-parallel
description: 'Parallel pipeline execution of bmad-bmm-dev-story. Use when the user says "parallel dev story", "fast dev story", "parallel implement", or wants to speed up story development.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/commands/bmad-bmm-dev-story.md` and follow its instructions EXACTLY
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

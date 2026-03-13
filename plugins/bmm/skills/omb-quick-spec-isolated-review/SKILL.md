---
name: omb-quick-spec-isolated-review
description: 'Quick Spec with isolated adversarial review. Use when the user says "isolated review quick spec", "unbiased review quick spec", or wants spec review by a separate context-free agent.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-quick-spec/SKILL.md` and follow its instructions EXACTLY
2. When workflow.md reaches step file execution, execute step-01 through step-03 as normal
3. In step-04, execute Sections 1–3 as normal (present spec, handle feedback, finalize)
4. In step-04 Section 4 Final Menu, when the user selects [R] Adversarial Review, apply the **isolated review override** below instead of the inline `<invoke-task>` process
5. After the override completes, return to step-04 Section 4 Final Menu and redisplay it

</steps>

## Isolated Review Override

When the user selects [R] in step-04 Section 4 Final Menu, do NOT execute the inline `<invoke-task>` review. Instead:

### 1. Resolve Inputs

Read `{finalFile}` completely and capture its full content as `{spec_content}`.

Check if `{project-root}/_bmad/project-context.md` exists. If it does, capture its path as `{project_context_path}`.

### 2. Spawn Isolated Reviewer

#### Teammate Persona

Read `references/personas/spec-reviewer.yaml` (relative to the skill root) and include its `persona` block at the top of the agent prompt.

#### Agent Spawn

Spawn a **foreground** agent (model: **opus**) with the following prompt:

```
{persona from references/personas/spec-reviewer.yaml}

You are an isolated adversarial reviewer. You have NO context about the conversations,
codebase investigation, or reasoning that produced this spec. You judge only what the
spec document shows.

## Your Task

Read `{project-root}/_bmad/core/tasks/bmad-review-adversarial-general/workflow.md` and execute it fully, in order.

## Inputs

### content (the spec to review)

{spec_content — include the entire spec inline here}

### also_consider (optional, only if project-context.md exists)

Read `{project_context_path}` for project coding standards and patterns to check against.

## Output

Return your findings as a Markdown list with descriptions only, as specified by the task.
```

**Included in the prompt:**
- Spec reviewer persona
- `{spec_content}` in full (literal content, not a reference)
- Path to `bmad-review-adversarial-general/workflow.md` (agent reads it directly)
- Path to `project-context.md` (if it exists)

**Excluded from the prompt (context isolation):**
- Conversation history and discovery session notes from step-01
- Codebase investigation results from step-02
- User discussions, clarifications, and feedback from step-04 Sections 1–2
- Any rationale or decisions made during spec generation

### 3. Process Findings

Receive the agent's findings and process them:

- **If zero findings:** HALT — this is suspicious. Re-analyze or request user guidance.
- Evaluate severity (Critical, High, Medium, Low) and validity (real, noise, undecided).
- DO NOT exclude findings based on severity or validity unless explicitly asked.
- Order findings by severity.
- Number the ordered findings (F1, F2, F3, etc.).
- If TodoWrite or similar tool is available, create a TODO per finding with ID, severity, validity, and description; otherwise present findings as a table with columns: ID, Severity, Validity, Description.

### 4. Return to Final Menu

Return to step-04 Section 4 Final Menu and redisplay it for the user to continue.

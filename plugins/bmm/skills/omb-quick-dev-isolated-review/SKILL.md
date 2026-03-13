---
name: omb-quick-dev-isolated-review
description: 'Quick Dev with isolated adversarial review. Use when the user says "isolated review quick dev", "unbiased review quick dev", or wants implementation review by a separate context-free agent.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-quick-dev/SKILL.md` and follow its instructions EXACTLY
2. When workflow.md reaches step file execution, execute step-01 through step-04 as normal
3. After step-04 completes, apply the **isolated review override** below instead of normal step-05
4. After the override completes, load and follow step-06-resolve-findings.md as normal

</steps>

## Isolated Review Override

When step-04 completes, do NOT load step-05-adversarial-review.md. Instead, perform the following:

### 1. Construct Diff

Build complete diff of all changes since workflow started, following the same logic as step-05 section "1. Construct Diff":

**If `{baseline_commit}` is a Git commit hash:**

Tracked file changes:

```bash
git diff {baseline_commit}
```

New untracked files: only files YOU created during steps 2-4. For each, include full content as a "new file" addition.

**If `{baseline_commit}` is "NO_GIT":**

Use best-effort diff construction — list modified files and their changes, include new files with full content.

Merge all changes into `{diff_output}`. Do NOT `git add` anything.

### 2. Resolve Project Context

Check if `{project-root}/_bmad/project-context.md` exists. If it does, capture its path as `{project_context_path}`.

### 3. Spawn Isolated Reviewer

#### Teammate Persona

Read `references/personas/adversarial-reviewer.yaml` (relative to the skill root) and include its `persona` block at the top of the agent prompt.

#### Agent Spawn

Spawn a **foreground** agent (model: **opus**) with the following prompt:

```
{persona from references/personas/adversarial-reviewer.yaml}

You are an isolated adversarial reviewer. You have NO context about why this code was written,
what alternatives were considered, or what the implementation plan was. You judge only what the diff shows.

## Your Task

Read `{project-root}/_bmad/core/tasks/bmad-review-adversarial-general/workflow.md` and execute it fully, in order.

## Inputs

### content (the diff to review)

{diff_output — include the entire diff inline here}

### also_consider (optional, only if project-context.md exists)

Read `{project_context_path}` for project coding standards and patterns to check against.

## Output

Return your findings as a Markdown list with descriptions only, as specified by the task.
```

**Included in the prompt:**
- Adversarial reviewer persona
- `{diff_output}` in full (literal content, not a reference)
- Path to `bmad-review-adversarial-general/workflow.md` (agent reads it directly)
- Path to `project-context.md` (if it exists)

**Excluded from the prompt (context isolation):**
- Implementation plans and design decisions
- Step-02 context gathering results
- Step-04 self-check results
- Task lists, acceptance criteria details, tech-spec content

### 4. Process Findings

Receive the agent's findings and process them following the same logic as step-05 section "3. Process Findings":

- **If zero findings:** HALT — this is suspicious. Re-analyze or request user guidance.
- Evaluate severity (Critical, High, Medium, Low) and validity (real, noise, undecided).
- DO NOT exclude findings based on severity or validity unless explicitly asked.
- Order findings by severity.
- Number the ordered findings (F1, F2, F3, etc.).
- If TodoWrite or similar tool is available, create a TODO per finding with ID, severity, validity, and description; otherwise present findings as a table with columns: ID, Severity, Validity, Description.

### 5. Resume Normal Workflow

Load and follow `{project-root}/_bmad/bmm/workflows/bmad-quick-flow/quick-dev/steps/step-06-resolve-findings.md` as normal.

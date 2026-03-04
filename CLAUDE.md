# Parallel Pipeline Skill Development Guide

Patterns and principles for writing skills that parallelize bmad workflows using agent teams.

## Core Principle: Never Duplicate the Original Workflow

A skill **references** the original command/workflow and **overrides** only the steps being parallelized. Do not rewrite workflow logic inside the skill.

## SKILL.md Structure

### 1. Critical Steps — Reference the Original Command

```markdown
<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/commands/{command}.md` and follow its instructions EXACTLY
2. When workflow.xml reaches instructions.xml execution, execute steps {sequential range} as normal
3. After step {N}, apply the **parallel override** below instead of normal step {parallel target}
4. After parallel work completes, resume sequential execution from instructions.xml step {resume point} onward

</steps>
```

- Follow the original command as-is, overriding only specific steps
- Steps before and after the override execute normally
- Always state where to resume after the parallel work completes

### 2. Base Context Block — Propagate Preceding Step Outputs

```markdown
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- story_key: {{story_key}}
- ...

## Step {N} Outputs (pass to all teammates)
- {outputs produced by preceding steps}
```

Preceding step outputs must be passed to teammates. This includes not just workflow variables but also **plans and analysis results** that the steps produced.

### 3. Phase Structure

| Phase | Purpose |
|---|---|
| Phase 1: Team Setup | Create team and tasks with dependencies |
| Phase 2: Parallel Execution | Spawn agents and execute work |
| Phase 3+: Post-processing | Aggregate results, validate, etc. |
| Final Phase: Cleanup | Shut down teammates and clean up the team |

### 4. Pipeline Selection — Only When Branching Exists

When there are 2+ branches, use SKILL.md as a router and split into `references/`.

```markdown
### Pipeline Selection

- **Condition A**: Read and follow `references/pipeline-a.md`
- **Condition B**: Read and follow `references/pipeline-b.md`
```

When there is no branching, keep everything in a single SKILL.md.

## Agent Prompt Patterns

### Spawn Format

```markdown
Spawn **{agent-name}** as a background teammate (model: **{model}**)
```

- Specify the model only in the spawn header. Do not create a separate "Model Selection" section.
- Mechanical tasks → sonnet, judgment-heavy tasks → opus

### Prompt Structure

```
You are a team member of team "{team-name}".
Your task: {task-name}

Read {installed_path}/instructions.xml and execute the following for {scope} ONLY:
- {step/section reference to execute}

{base context — includes step outputs}

When complete, mark {task-name} as completed and report to team-lead with:
- {report items}
```

**Do not:**
- Copy checklists or logic from instructions.xml into the prompt
- Duplicate content that the agent can read directly

**Do:**
- Specify exactly which section of instructions.xml to execute
- Pass preceding step outputs via base context
- List specific items to report on completion

## Anti-Patterns

### Do Not Use Tool-Specific Terminology

```markdown
# Bad
Create tracked tasks using `TaskCreate`. Store returned numeric task IDs for `blockedBy` references.

# Good
Create tracked tasks for each pipeline stage, with dependencies reflecting the stage order.
```

The team lead already knows how to use the tools. The skill describes **what** to do, not **how**.

### Shared Conventions Only When There Are Multiple Pipelines

Only add a Shared Conventions section when two or more pipelines share common rules. It is unnecessary for a single pipeline.

Also, do not repeat content in Shared Conventions that is already described in the reference files.

### Do Not Duplicate Workflow Content

The skill's role is to describe the **parallelization strategy**, not to rewrite the original workflow. Agents can read instructions.xml directly.

## Reference Examples

- `plugins/bmm/skills/omb-bmm-dev-story-parallel/` — 4-stage TDD pipeline, 2 branches (implementation + review continuation)

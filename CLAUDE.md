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

### Teammate Personas

Define teammate personas as YAML files in `references/personas/` following the BMAD agent persona format:

```yaml
persona:
  role: Short role title — pipeline phase context
  identity: >
    1-2 sentences. What this teammate does and how it approaches the work.
  communication_style: "One sentence. Tone and reporting style."
  principles: |
    - Responsible for: positive scope (what this teammate owns)
    - Not responsible for: negative scope (what belongs to other pipeline stages)
    - Any additional behavioral constraint specific to this role
```

**Key rules:**
- `role`: imperative title + pipeline phase (e.g., "Test-first specialist — Red phase of the TDD pipeline")
- `communication_style`: follows BMAD convention — a single quoted sentence defining tone
- `principles`: always include both positive and negative scope to prevent agents from wandering into other stages' territory
- One persona file per distinct role. Roles shared across pipelines (e.g., `validator.yaml`) reuse the same file.

### Prompt Structure

```
{persona from references/personas/{role}.yaml}
You are {agent-name} of team "{team-name}".
Your task: {task-name}

Read {installed_path}/instructions.xml and execute the following for {scope} ONLY:
- {step/section reference to execute}

{base context — includes step outputs}

When complete, mark {task-name} as completed and report to team-lead with:
- {report items}
```

Pipeline files must include a "Teammate Personas" section instructing the team lead to read and embed persona content:

```markdown
## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt,
read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.
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

## Documentation

When creating or modifying a skill, update the plugin's `README.md` (e.g., `plugins/{plugin}/README.md`) with a matching entry. Follow the existing format: linked heading, one-line description, agent table, and a brief explanation of the skill's approach.

## Reference Examples

- `plugins/bmm/skills/omb-bmm-dev-story-parallel/` — 4-stage TDD pipeline, 2 branches (implementation + review continuation)
- `plugins/bmm/skills/omb-bmm-quick-dev-isolated-review/` — single-agent context isolation (no team, foreground sub-agent)

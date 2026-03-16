---
name: omb-create-story-parallel
description: 'Parallelized version of bmad-create-story that uses an Agent Team for concurrent research steps. Steps 2, 3, 4 run in parallel, and teammates stay alive for follow-up refinement during step 5+. Use when the user says "parallel create story", "fast create story", "parallel story", or wants to speed up story creation.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-create-story/SKILL.md` and follow its instructions EXACTLY
2. When workflow.xml reaches instructions.xml execution, execute step 1 (determine target story) as normal
3. For instructions.xml steps 2, 3, 4: apply the **parallel execution override** below instead of sequential execution
4. After all parallel agents complete, resume sequential execution from instructions.xml step 5 onward
5. When the user requests additional investigation on a specific area, route it to the relevant teammate instead of doing it yourself
6. After step 6 completes, shut down teammates and clean up the team

</steps>

## Parallel Execution Override

After completing instructions.xml step 1, prepare a base context block:

```
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- planning_artifacts: {resolved}
- implementation_artifacts: {resolved}
- epic_num: {{epic_num}}
- story_num: {{story_num}}
- story_key: {{story_key}}
- story_id: {{story_id}}
- story_title: {{story_title}}
- sprint_status: {resolved}
- communication_language: {resolved}
- document_output_language: {resolved}

## Step 1 Outputs
- Target story identifier, title, and description from the epic list
- Epic context and story position within the epic
- Any constraints or prerequisites identified during story selection
```

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase 1: Team Setup

Create a team named `story-{{story_key}}`.

Create tracked tasks for each research dimension, plus an aggregate task that depends on all three completing:
- **artifact-analysis**: "Planning artifact review"
- **architecture-analysis**: "Codebase pattern analysis"
- **tech-research**: "Technical approach research"
- **aggregate**: "Synthesize research results for story drafting" (blocked by the above three)

## Phase 2: Parallel Research

Spawn ALL 3 teammates in a single message:

Spawn **artifact-analyst** as a background teammate (model: **opus**)

```
{persona from references/personas/artifact-analyst.yaml}
You are artifact-analyst of team "story-{{story_key}}".
Your task: artifact-analysis

Read {installed_path}/instructions.xml and execute the following ONLY:
- <step n="2">

{base context}

When complete, mark artifact-analysis as completed and report to team-lead with:
- Key requirements and acceptance criteria extracted
- Dependencies and constraints discovered
- Cross-references to other stories or epics
Then go idle — you may receive follow-up requests for deeper investigation.
```

Spawn **architecture-analyst** as a background teammate (model: **sonnet**)

```
{persona from references/personas/architecture-analyst.yaml}
You are architecture-analyst of team "story-{{story_key}}".
Your task: architecture-analysis

Read {installed_path}/instructions.xml and execute the following ONLY:
- <step n="3">

{base context}

When complete, mark architecture-analysis as completed and report to team-lead with:
- Relevant codebase patterns and conventions identified
- Technical constraints and integration points
- Recommended approach based on existing architecture
Then go idle — you may receive follow-up requests for deeper investigation.
```

Spawn **tech-researcher** as a background teammate (model: **sonnet**)

```
{persona from references/personas/tech-researcher.yaml}
You are tech-researcher of team "story-{{story_key}}".
Your task: tech-research

Read {installed_path}/instructions.xml and execute the following ONLY:
- <step n="4">

Since step 3 (architecture analysis) runs concurrently, you cannot receive its output directly.
To identify technologies for research, load the architecture document from {planning_artifacts} yourself
and extract relevant libraries, frameworks, and APIs before proceeding with step 4's research tasks.

{base context}

When complete, mark tech-research as completed and report to team-lead with:
- Technologies and libraries evaluated with trade-offs
- Recommended technical approaches
- Implementation risks and mitigations identified
Then go idle — you may receive follow-up requests for deeper investigation.
```

## Phase 3: Sequential Resumption

After all 3 teammates complete (aggregate task unblocks), use their analyses as input context for instructions.xml step 5 onward.

If the user requests additional investigation on a specific area during step 5+:
- Route the request to the relevant teammate (e.g., architecture questions → **architecture-analyst**)
- The teammate retains its original research context and can dig deeper immediately

## Final Phase: Cleanup

After step 6 completes, shut down all teammates and clean up the team and its task list.

## Next Step Handoff

After cleanup, use **AskUserQuestion** to present next steps.

**If TEA is available** (`/omb-tea-testarch-atdd-parallel` skill exists):

**Question:** "Story creation complete. Proceed to ATDD test design?"

**Options:**
1. **Proceed to ATDD** — Run `/omb-tea-testarch-atdd-parallel`
2. **Continue reviewing** — Stay in current context for further review
3. **Done for now** — End pipeline

**If TEA is unavailable:**

**Question:** "Story creation complete. Proceed to Dev Story implementation?"

**Options:**
1. **Proceed to Dev Story** — Run `/omb-dev-story-parallel`
2. **Continue reviewing** — Stay in current context for further review
3. **Done for now** — End pipeline

If user selects the first option, invoke the corresponding slash command.

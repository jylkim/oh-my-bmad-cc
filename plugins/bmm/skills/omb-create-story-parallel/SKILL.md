---
name: omb-create-story-parallel
description: 'Parallelized version of bmad-create-story that uses an Agent Team for concurrent research steps. Includes scope discovery after artifact analysis to narrow development goals before parallel research. Use when the user says "parallel create story", "fast create story", "parallel story", or wants to speed up story creation.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/skills/bmad-create-story/SKILL.md` and follow its instructions EXACTLY
2. When workflow.xml reaches instructions.xml execution, execute step 1 (determine target story) as normal
3. Execute instructions.xml step 2 (artifact analysis) as normal — team lead does this directly
4. After step 2, run **Scope Discovery** below to narrow development goals with the user
5. After scope discovery, apply the **parallel execution override** below instead of sequential steps 3, 4
6. After all parallel agents complete, resume sequential execution from instructions.xml step 5 onward
7. When the user requests additional investigation on a specific area, route it to the relevant teammate instead of doing it yourself
8. After step 6 completes, shut down teammates and clean up the team

</steps>

## Scope Discovery

After completing instructions.xml step 2, review the extracted requirements, constraints, and dependencies. Assess whether the story scope is clear enough to proceed directly to research.

**Signals that scope is clear — skip to parallel research:**
- Story has specific, unambiguous acceptance criteria
- Dependencies and constraints are well-defined
- Development approach is obvious from the artifacts

**Signals that scope needs narrowing:**
- Multiple reasonable interpretations of what to build
- Acceptance criteria are vague or overly broad
- Trade-offs exist that affect the research direction

When scope needs narrowing, **ask the user interactively** — present one question at a time, wait for their explicit response, then proceed to the next question or move on. Do not batch questions or continue without the user's answer:

- Ask **one question at a time** — do not overwhelm with multiple questions
- **Prefer multiple choice** when natural options exist (e.g., "Should this story cover: (a) only the API layer, (b) API + basic UI, or (c) full stack including tests?")
- **Start broad, then narrow** — confirm the core intent first, then refine constraints
- **Validate assumptions explicitly** — "The PRD mentions X, but the epic description says Y. Which takes priority?"
- **Ground questions in artifact analysis results** — reference specific requirements, dependencies, or ambiguities discovered in step 2
- **Always lead with context, then recommend** — before asking a question, briefly explain what you found during artifact analysis and why this decision matters for the upcoming research. Then state your recommendation with reasoning before presenting options. Example: "The PRD lists both REST and GraphQL as possible API styles. Since the existing codebase uses REST exclusively, I'd recommend (a) REST to stay consistent — but (b) GraphQL could be worth it if the frontend needs flexible queries. Which do you prefer?"

**Exit condition:** Proceed when development goals are clear enough to guide research, OR the user says "proceed" / "let's move on".

Capture the confirmed scope as a concise summary (3-5 bullet points) for inclusion in the base context.

## Parallel Execution Override

After scope discovery, prepare a base context block:

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

## Step 1-2 Outputs
- Target story identifier, title, and description from the epic list
- Epic context and story position within the epic
- Key requirements and acceptance criteria extracted from planning artifacts
- Dependencies and constraints discovered

## Scope Discovery Outputs
- Confirmed development goals (the 3-5 bullet summary)
- Any scope exclusions or deferrals agreed with the user
- Priority trade-offs resolved during discovery
```

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase 1: Team Setup

Create a team named `story-{{story_key}}`.

Create tracked tasks for each research dimension, plus an aggregate task that depends on both completing:
- **architecture-analysis**: "Codebase pattern analysis"
- **tech-research**: "Technical approach research"
- **aggregate**: "Synthesize research results for story drafting" (blocked by the above two)

## Phase 2: Parallel Research

Spawn BOTH teammates in a single message:

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

## Phase 3: Story Drafting

After both teammates complete (aggregate task unblocks), use their analyses as input context for instructions.xml step 5 onward.

If the user requests additional investigation on a specific area during step 5+:
- Route the request to the relevant teammate (e.g., architecture questions → **architecture-analyst**)
- The teammate retains its original research context and can dig deeper immediately

## Final Phase: Cleanup

After step 6 completes, shut down all teammates and clean up the team and its task list.


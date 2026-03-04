---
name: omb-bmm-create-story-parallel
description: 'Parallelized version of bmad-bmm-create-story that uses an Agent Team for concurrent research steps. Steps 2, 3, 4 run in parallel, and teammates stay alive for follow-up refinement during step 5+. Use when the user says "parallel create story", "fast create story", "parallel story", or wants to speed up story creation.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/commands/bmad-bmm-create-story.md` and follow its instructions EXACTLY
2. When workflow.xml reaches instructions.xml execution, execute step 1 (determine target story) as normal
3. For instructions.xml steps 2, 3, 4: apply the **parallel execution override** below instead of sequential execution
4. After all parallel agents complete, resume sequential execution from instructions.xml step 5 onward
5. When the user requests additional investigation on a specific area, route it to the relevant teammate instead of doing it yourself
6. After step 6 completes, shut down teammates and clean up the team

</steps>

## Parallel Execution Override

After completing instructions.xml step 1, prepare a base context block with all resolved workflow variables:

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
```

### Team Setup

Create a team named `story-{{story_key}}`.

### Parallel Research

Spawn ALL 3 teammates in a SINGLE message (`run_in_background: true`):

**artifact-analyst** — Step 2, model: **opus**

```
You are a team member of team "story-{{story_key}}".

Read {installed_path}/instructions.xml and execute <step n="2"> exactly as written.
Resolve all references using the workflow context below.

{base context}

When complete, send your analysis to team-lead.
Then go idle — you may receive follow-up requests for deeper investigation.
```

**architecture-analyst** — Step 3, model: **sonnet**

```
You are a team member of team "story-{{story_key}}".

Read {installed_path}/instructions.xml and execute <step n="3"> exactly as written.
Resolve all references using the workflow context below.

{base context}

When complete, send your analysis to team-lead.
Then go idle — you may receive follow-up requests for deeper investigation.
```

**tech-researcher** — Step 4, model: **sonnet**

```
You are a team member of team "story-{{story_key}}".

Read {installed_path}/instructions.xml and execute <step n="4"> exactly as written.
Resolve all references using the workflow context below.

{base context}

When complete, send your analysis to team-lead.
Then go idle — you may receive follow-up requests for deeper investigation.
```

### Resuming Sequential Execution

After all 3 teammates send their analyses, use those results as input context for instructions.xml step 5 onward.

If the user requests additional investigation on a specific area during step 5+:
- Route the request to the relevant teammate (e.g., architecture questions → **architecture-analyst**)
- The teammate retains its original research context and can dig deeper immediately

### Cleanup

After step 6 completes, request all teammates to shut down gracefully and clean up the team and its task list.

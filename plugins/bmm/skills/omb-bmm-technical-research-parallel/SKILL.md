---
name: omb-bmm-technical-research-parallel
description: 'Parallelized version of bmad-bmm-technical-research that uses an Agent Team for concurrent research steps. Steps 2-5 run in parallel with 4 specialist agents, with follow-up deep-dives available before user approval. Use when the user says "parallel technical research", "fast technical research", "parallel research", or wants to speed up technical research.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/commands/bmad-bmm-technical-research.md` and follow its instructions EXACTLY
2. Follow the workflow through CONFIGURATION, QUICK TOPIC DISCOVERY, and ROUTE TO TECHNICAL RESEARCH STEPS — execute step 1 (scope confirmation) as normal
3. After step 1 completes, apply the **parallel execution override** below instead of loading step-02 sequentially
4. When the user requests additional investigation on a specific area before [C] approval, route it to the relevant teammate instead of doing it yourself
5. After all agents write to the document, shut down teammates, clean up the team, and resume with step 6 (synthesis)

</steps>

## Parallel Execution Override

After completing step 1, prepare a base context block with all resolved workflow variables:

```
## Workflow Context
- project-root: {resolved}
- planning_artifacts: {resolved}
- research_topic: {{research_topic}}
- research_goals: {{research_goals}}
- date: {{date}}
- communication_language: {resolved}
- document_output_language: {resolved}
- output_file: {resolved path to the research document created in ROUTE TO TECHNICAL RESEARCH STEPS}
- steps_path: {project-root}/_bmad/bmm/workflows/1-analysis/research/technical-steps

## Step 1 Outputs
- Confirmed research scope and approach
- Research document created at output_file with template contents
- Scope confirmation section already appended to document

## Required Reading
Before starting research, read the scope confirmation section in output_file to understand
the confirmed research scope and any user refinements from the Step 1 conversation.
```

### Team Setup

Create a team named `tech-research`.

### Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

### Parallel Research

Spawn ALL 4 teammates in a single message as background agents:

**tech-stack-analyst** — Step 2, model: **sonnet**

```
{persona from references/personas/tech-stack-analyst.yaml}
You are tech-stack-analyst of team "tech-research".
Your task: Technology Stack Analysis

Read {steps_path}/step-02-technical-overview.md and conduct the Technology Stack Analysis research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

**integration-analyst** — Step 3, model: **sonnet**

```
{persona from references/personas/integration-analyst.yaml}
You are integration-analyst of team "tech-research".
Your task: Integration Patterns Analysis

Read {steps_path}/step-03-integration-patterns.md and conduct the Integration Patterns research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

**architecture-analyst** — Step 4, model: **sonnet**

```
{persona from references/personas/architecture-analyst.yaml}
You are architecture-analyst of team "tech-research".
Your task: Architectural Patterns Analysis

Read {steps_path}/step-04-architectural-patterns.md and conduct the Architectural Patterns research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

**implementation-researcher** — Step 5, model: **sonnet**

```
{persona from references/personas/implementation-researcher.yaml}
You are implementation-researcher of team "tech-research".
Your task: Implementation Research

Read {steps_path}/step-05-implementation-research.md and conduct the Implementation Research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

### Content Assembly

After all 4 teammates report their key findings summaries:

1. Present the combined key findings to the user with [C] Continue
2. If the user requests additional investigation before approving, route to the relevant teammate (e.g., architecture questions → **architecture-analyst**) — the teammate retains its original research context and can dig deeper immediately
3. If the user approves, relay approval to each teammate in step order (2 → 3 → 4 → 5) so they write their content to the document
4. After all writes complete, update frontmatter: `stepsCompleted: [1, 2, 3, 4, 5]`
5. Shut down all teammates and clean up the team

### Resuming Sequential Execution

After cleanup, load step-06-research-synthesis.md and execute it normally to produce the final comprehensive document.

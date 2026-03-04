---
name: omb-bmm-market-research-parallel
description: 'Parallelized version of bmad-bmm-market-research that uses an Agent Team for concurrent research steps. Steps 2-5 run in parallel with 4 specialist agents, with follow-up deep-dives available before user approval. Use when the user says "parallel market research", "fast market research", "parallel research", or wants to speed up market research.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/commands/bmad-bmm-market-research.md` and follow its instructions EXACTLY
2. Follow the workflow through CONFIGURATION, QUICK TOPIC DISCOVERY, and ROUTE TO MARKET RESEARCH STEPS — execute step 1 (scope confirmation) as normal
3. After step 1 completes, apply the **parallel execution override** below instead of loading step-02 sequentially
4. When the user requests additional investigation on a specific area before [C] approval, route it to the relevant teammate instead of doing it yourself
5. After all agents write to the document, shut down teammates, clean up the team, and resume with step 6 (research completion)

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
- output_file: {resolved path to the research document created in ROUTE TO MARKET RESEARCH STEPS}
- steps_path: {project-root}/_bmad/bmm/workflows/1-analysis/research/market-steps

## Step 1 Outputs
- Confirmed research scope and approach
- Research document created at output_file with template contents
- Scope confirmation section already appended to document
```

### Team Setup

Create a team named `market-research`.

### Parallel Research

Spawn ALL 4 teammates in a single message as background agents:

**customer-behavior-analyst** — Step 2, model: **sonnet**

```
You are a team member of team "market-research".
Your task: Customer Behavior and Segments Analysis

Read {steps_path}/step-02-customer-behavior.md and conduct the Customer Behavior and Segments research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

**pain-points-analyst** — Step 3, model: **sonnet**

```
You are a team member of team "market-research".
Your task: Customer Pain Points and Needs Analysis

Read {steps_path}/step-03-customer-pain-points.md and conduct the Customer Pain Points and Needs research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

**decision-journey-analyst** — Step 4, model: **sonnet**

```
You are a team member of team "market-research".
Your task: Customer Decisions and Journey Analysis

Read {steps_path}/step-04-customer-decisions.md and conduct the Customer Decisions and Journey research.
Do the web searches specified in the step and produce the content sections as defined in "Content Structure".

Do not present [C] Continue to the user directly — the team lead will coordinate user review on your behalf.

{base context}

When complete, report to team-lead with:
- Key findings summary (3-5 bullet points)
- A request for user [C] Continue approval

Then go idle — you may receive follow-up requests for deeper investigation before user approval.
After team-lead relays user approval, append your full research content to the output_file and report completion.
```

**competitive-analyst** — Step 5, model: **sonnet**

```
You are a team member of team "market-research".
Your task: Competitive Analysis

Read {steps_path}/step-05-competitive-analysis.md and conduct the Competitive Analysis research.
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
2. If the user requests additional investigation before approving, route to the relevant teammate (e.g., competitive questions → **competitive-analyst**) — the teammate retains its original research context and can dig deeper immediately
3. If the user approves, relay approval to each teammate in step order (2 → 3 → 4 → 5) so they write their content to the document
4. After all writes complete, update frontmatter: `stepsCompleted: [1, 2, 3, 4, 5]`
5. Shut down all teammates and clean up the team

### Resuming Sequential Execution

After cleanup, load step-06-research-completion.md and execute it normally to produce the final comprehensive document.

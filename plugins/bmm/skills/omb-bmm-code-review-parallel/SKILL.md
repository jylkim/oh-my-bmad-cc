---
name: omb-bmm-code-review-parallel
description: 'Parallel pipeline execution of bmad-bmm-code-review. Use when the user says "parallel code review", "fast code review", "parallel review", or wants to speed up adversarial code review.'
---

<steps CRITICAL="TRUE">

1. Read `{project-root}/.claude/commands/bmad-bmm-code-review.md` and follow its instructions EXACTLY
2. When workflow.xml reaches instructions.xml execution, execute steps 1-2 (setup) as normal
3. After step 2, apply the **parallel review override** below instead of normal step 3
4. After parallel review completes with aggregated findings, resume sequential execution from instructions.xml step 4 onward

</steps>

## Parallel Review Override

After completing instructions.xml step 2, prepare a base context block with all resolved workflow variables:

```
## Workflow Context
- project-root: {resolved}
- installed_path: {resolved}
- implementation_artifacts: {resolved}
- story_key: {{story_key}}
- story_path: {{story_path}}
- project_context: {resolved}
- sprint_status: {resolved}

## Step 1-2 Outputs (pass to all teammates)
- comprehensive_file_list: [union of story File List + git discovered files]
- git_discrepancies: [story File List vs git reality, from step 1]
- review_attack_plan: [step 2 output — extracted ACs, tasks with [x]/[ ] status, claimed changes, review plan]
```

### Phase 1: Team Setup

Create a team named `code-review-{{story_key}}`.

### Teammate Personas

Persona files are in `references/personas/` (relative to this skill's directory). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

Create tracked tasks for each review dimension. The first 4 are independent. The aggregate task depends on all 4 completing.

- **git-audit**: "Cross-reference git changes vs story File List"
- **ac-validate**: "Verify all Acceptance Criteria against implementation"
- **task-audit**: "Verify all [x] task completion claims"
- **code-quality**: "Deep code quality review on all implementation files"
- **aggregate**: "Aggregate and categorize all review findings" (blocked by the above 4)

### Phase 2: Parallel Review

All 4 review dimensions are independent — spawn ALL reviewer agents in a SINGLE message.

---

#### Dimension A — Git Discrepancy Audit

Spawn **git-auditor** as a background teammate (model: **sonnet**)

```
{persona from references/personas/git-auditor.yaml}
You are git-auditor of team "code-review-{{story_key}}".
Your task: git-audit

Read {installed_path}/instructions.xml and execute the following from step 3 ONLY:
- "Git vs Story Discrepancies" review actions

{base context — includes review_attack_plan, comprehensive_file_list, git_discrepancies}

When complete, mark git-audit as completed and report to team-lead with:
- All findings with severity and evidence
- Summary count by severity
```

---

#### Dimension B — Acceptance Criteria Validation

Spawn **ac-validator** as a background teammate (model: **opus**)

```
{persona from references/personas/ac-validator.yaml}
You are ac-validator of team "code-review-{{story_key}}".
Your task: ac-validate

Read {installed_path}/instructions.xml and execute the following from step 3 ONLY:
- "AC Validation" review actions

{base context — includes review_attack_plan, comprehensive_file_list}

When complete, mark ac-validate as completed and report to team-lead with:
- Per-AC status (IMPLEMENTED / PARTIAL / MISSING)
- All findings with severity and evidence
- Summary count by severity
```

---

#### Dimension C — Task Completion Audit

Spawn **task-auditor** as a background teammate (model: **opus**)

```
{persona from references/personas/task-auditor.yaml}
You are task-auditor of team "code-review-{{story_key}}".
Your task: task-audit

Read {installed_path}/instructions.xml and execute the following from step 3 ONLY:
- "Task Completion Audit" review actions

{base context — includes review_attack_plan, comprehensive_file_list}

When complete, mark task-audit as completed and report to team-lead with:
- Per-task verification (DONE / PARTIAL / NOT_DONE)
- All findings with severity and evidence
- Summary count by severity
```

---

#### Dimension D — Code Quality Deep Dive

Spawn **code-reviewer** as a background teammate (model: **opus**)

```
{persona from references/personas/code-reviewer.yaml}
You are code-reviewer of team "code-review-{{story_key}}".
Your task: code-quality

Read {installed_path}/instructions.xml and execute the following from step 3 ONLY:
- "Code Quality Deep Dive" review actions
- Also read the minimum issue check at the end of step 3

{base context — includes review_attack_plan, comprehensive_file_list}

When complete, mark code-quality as completed and report to team-lead with:
- Per-file findings with severity, category, and evidence
- Summary count by severity and category
```

---

### Phase 3: Findings Aggregation

When all 4 reviewers complete (aggregate task unblocks):

1. Collect findings from all agents' reports
2. Deduplicate overlapping findings (e.g., git-auditor and task-auditor may flag same file)
3. Count totals by severity: CRITICAL, HIGH, MEDIUM, LOW
4. If total findings < 3, send **code-reviewer** a message requesting deeper analysis. Wait for additional findings.
5. Mark aggregate as completed

Pass the aggregated, categorized findings to instructions.xml step 4.

### Phase 4: Completion & Cleanup

After instructions.xml step 5 completes:

Request all teammates to shut down gracefully.
Clean up the team and its task list.

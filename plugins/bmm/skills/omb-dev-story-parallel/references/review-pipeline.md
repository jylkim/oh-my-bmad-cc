# Review Follow-up Pipeline (Phase R1-R5)

Lightweight 2-stage pipeline for addressing `[AI-Review]` action items after code review. All implementation tasks are already complete — only review findings need to be fixed.

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase R1: Review Item Analysis

1. **Extract all unchecked `[AI-Review]` items** from "Tasks/Subtasks → Review Follow-ups (AI)" section
2. **For each item, identify**:
   - Severity (High/Med/Low)
   - Target files to modify
   - Related acceptance criteria or review finding
3. **Build dependency graph** based on file overlap:
   - Items touching different files → independent → parallel
   - Items touching the same file(s) → dependent → sequential
4. **Output the execution plan** before proceeding (team lead decides final ordering):
   ```
   Review Items: 5 total (2 High, 2 Med, 1 Low)
   Parallel:   [R1, R3, R5] (independent files)
   Sequential: [R2 → R4] (both modify src/middleware/retry.rs)
   ```

## Phase R2: Review Team Setup

Create a team named `dev-{{story_key}}-review`.

Create tracked tasks for each review item's lightweight pipeline, with dependencies reflecting the stage order.

For each review item R:
- **fix-{R}**: "Fix review item {R}: [{severity}] {description}"
- **verify-{R}**: "Validate review fix {R}" (blocked by fix-{R})

For file-overlapping items (R depends on Q):
- fix-{R} is blocked by verify-{Q}

Final task:
- **integration-validate** (blocked by all verify-{R} tasks)

## Phase R3: Lightweight Fix Pipelines

**One review item, one agent, one pipeline stage.** Independent items launch concurrently.

---

### Stage A — Fix: Refactorer Addresses Review Finding

Spawn **refactorer-{R}** as a background teammate (model: **sonnet**)

```
{persona from references/personas/review-fixer.yaml}
You are refactorer-{R} of team "dev-{{story_key}}-review".
Your task: fix-{R}

Read {installed_path}/instructions.xml for project conventions reference.

Review finding to address:
- Severity: {severity}
- Description: {review item description}
- Related files: {target files}
- Related AC/context: {related acceptance criteria or reviewer comment}

Fix the issue while keeping ALL existing tests green.
- If the fix requires new tests, add them
- If existing tests need adjustment due to the fix, modify them with clear justification
- Run tests after every change to ensure no regressions

{base context}

When complete, mark fix-{R} as completed and report to team-lead with:
- All files modified (with summary of changes per file)
- How the review finding was addressed
- Test runner output confirming ALL tests pass
```

---

### Stage B — Verify: Validator Checks Fix

When refactorer-{R} sends its completion message, **immediately** spawn **validator-{R}** as a background teammate (model: **sonnet**)

```
{persona from references/personas/validator.yaml}
You are validator-{R} of team "dev-{{story_key}}-review".
Your task: verify-{R}

Read {installed_path}/instructions.xml for validation reference.

Validate that the review finding has been properly addressed:
- Run full test suite — no regressions
- Verify the specific review finding is resolved
- Run linting and code quality checks
- Confirm the fix doesn't introduce new issues

Do NOT mark the task complete — report results to team-lead only.

{base context}

Review finding:
- Severity: {severity}
- Description: {review item description}

Changed files:
{file list from refactorer's message}

Send results to team-lead:
- PASS: test counts, confirmation that review finding is addressed, linting status
- FAIL: FULL error output, what remains unresolved
```

---

### Fix Loop

If validator-{R} reports FAIL:

1. Send fix request back to **refactorer-{R}** with the EXACT error output
2. When refactorer-{R} reports completion, spawn new **validator-{R}** to re-verify
3. Max 3 fix cycles — after that, team-lead investigates and fixes directly

---

### Per-Item Completion (Team Lead)

When validator-{R} reports PASS, the team-lead **immediately**:

1. Mark the `[AI-Review]` task checkbox `[x]` in "Tasks/Subtasks → Review Follow-ups (AI)" section
2. Find the **corresponding action item** in "Senior Developer Review (AI) → Action Items" section by matching description, and mark it `[x]`
3. Add to Dev Agent Record → Completion Notes: `"✅ Resolved review finding [{severity}]: {description}"`
4. Update File List with all modified files from this item's pipeline
5. Mark verify-{R} as completed
6. Shut down refactorer-{R} and validator-{R}
7. If this unblocks file-dependent review items → immediately launch their Fix stage

## Phase R4: Integration Validation

After ALL verify-{R} tasks are completed, spawn:

**integration-validator** as a background teammate (model: **opus**)

```
{persona from references/personas/integration-validator.yaml}
You are integration-validator of team "dev-{{story_key}}-review".
Your task: integration-validate

Run the full regression test suite for the entire project.
Verify ALL existing and new tests pass together.
Verify that ALL review action items have been properly addressed.
Check for unintended interactions between independently applied fixes.

{base context}

All review items and changed files:
{summary from all fix pipeline completions}

Review summary:
- Total items addressed: {count}
- By severity: {High/Med/Low counts}

Send results to team-lead: pass/fail with full details.
```

If integration-validator reports failures:
- Identify which review fix caused the issue
- Send fix request to the responsible **refactorer-{R}**
- After fix, re-run **validator-{R}**, then re-run **integration-validator**
- Max 3 cycles before manual intervention

## Phase R5: Review Completion & Cleanup

After integration validation passes:

1. Verify ALL `[AI-Review]` tasks are marked `[x]` in "Review Follow-ups (AI)" section
2. Verify ALL corresponding action items are marked `[x]` in "Senior Developer Review (AI) → Action Items" section
3. Add Change Log entry: `"Addressed code review findings — {resolved_count} items resolved (Date: {{date}})"`
4. Finalize File List, Dev Agent Record, and Change Log

Request all teammates to shut down gracefully.
Clean up the team and its task list.

Resume instructions.xml step 9 (story completion).

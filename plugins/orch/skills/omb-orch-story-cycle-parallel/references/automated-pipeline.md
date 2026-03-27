# Automated Pipeline

This document defines the automated execution pipeline invoked after the user selects a pipeline mode in the coordinator's Pipeline Gate.

## Execution Rules

<rules CRITICAL="TRUE">
1. **Coordinator executes all steps directly.** No teammates for Steps 1–5. The coordinator invokes each slash command itself.
2. **Coordinator NEVER writes or edits code.** All code changes must go through slash commands. Do not open editors, use Edit/Write tools, or spawn custom sub-agents to fix code directly.
3. **No custom sub-agents.** The only teammate the coordinator may spawn is the simplify-applier in Step 6. All other work goes through slash commands.
4. All steps run in **yolo** mode — no user interaction during automated steps.
5. **No commits until Step 7.** No `git commit`, `git add`, or staging operations during Steps 1–5.
6. **On failure:** Report which step failed and why, then STOP immediately. Do not continue.
7. **One step at a time.** Complete each step fully before starting the next.
8. **Rework routing is mandatory.** When the story status is `in-progress` after code review, the coordinator MUST follow the Rework Decision table — never skip rework or apply fixes directly regardless of how trivial the issues appear.
</rules>

## Step 1: ATDD Test Architecture

**Skip if `pipeline_mode` is `tea-excluded`.**

Coordinator directly runs: `/bmad-testarch-atdd {STORY_ID} yolo`

## Step 2: Dev Story Implementation

Coordinator directly runs: `/omb-dev-story-parallel {STORY_ID} yolo`

## Step 3: Test Automation

**Skip if `pipeline_mode` is `tea-excluded`.**

Coordinator directly runs: `/bmad-testarch-automate {STORY_ID} yolo`

## Step 4: Code Review

- **Rework remaining:**
  Coordinator directly runs: `/bmad-code-review {STORY_ID} yolo, create action items for all the issues`
- **Final iteration (no rework remaining):**
  Coordinator directly runs: `/bmad-code-review {STORY_ID} yolo, auto accept and fix all the issues`

## Rework Decision (after Step 4)

After each code review, read the story file and check story status:

| Story status | Rework remaining | Action |
|---|---|---|
| `done` | — | Proceed to Step 5 |
| `in-progress` | Yes | Re-run Step 2 (Dev Story) → Step 3 (Test Automation) → Step 4 (Code Review) |
| `in-progress` | No (final) | Proceed to Step 5 — remaining issues are deferred |

> When `pipeline_mode` is `tea-excluded`, rework skips Step 3 (Test Automation): Dev Story → Code Review only.

Maximum **3 total iterations** (initial run + up to 2 rework cycles).

## Step 5: Test Review

**Skip if `pipeline_mode` is `tea-excluded`.**

Coordinator directly runs: `/bmad-testarch-test-review {STORY_ID} yolo`

## Step 6: Simplify + Defer

The coordinator directly invokes `/simplify` (Phase 1–2 only: identify changes and launch three review agents).

After the review agents complete, classify each finding as **fix**, **defer**, or **reject**:

- **fix**: Spawn a **simplify-applier** teammate (model: **sonnet**) with the persona from `references/personas/simplify-applier.yaml`. Pass the approved fix items as context. The teammate applies each change atomically, running tests after each one. If tests fail, the teammate reverts the change and reclassifies it as **defer**.
- **defer**: Append to `{implementation_artifacts}/deferred-work.md` using the format below. If the file does not exist, create it with a `# Deferred Work` header first.
- **reject**: Discard — no action needed.

### Deferred Work Entry Format

```markdown
### {Finding Title}
- **Source:** Story {STORY_ID} — {STORY_TITLE}, Simplify review
- **File:** {file path}
- **Description:** {finding description}
- **Reason:** {deferral reason}
```

## Step 7: Commit

The coordinator directly performs the final commit:

1. Run `git status` and `git diff --stat` to review all changes from this pipeline run.
2. Present the summary to the user and ask for confirmation.
3. On approval, stage and commit with message: `feat: implement {STORY_TITLE} (Story {STORY_ID})`

## Pipeline Summary

After all steps complete, provide a summary of the full pipeline run:

- Total iterations executed
- Rework cycles taken (if any)
- Steps skipped (TEA-excluded steps, etc.)
- Deferred items (count and file location)
- Final story status

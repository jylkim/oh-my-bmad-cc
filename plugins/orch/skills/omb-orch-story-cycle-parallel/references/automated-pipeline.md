# Automated Pipeline

This document defines the automated execution pipeline invoked after the user selects a pipeline mode in the coordinator's Pipeline Gate.

## Execution Rules

<rules CRITICAL="TRUE">
1. **One teammate at a time.** Spawn the next teammate ONLY after the previous one completes successfully.
2. Each teammate runs in **yolo** mode — no user interaction during automated steps.
3. **Teammates MUST NOT commit.** No `git commit`, `git add`, or staging operations. Only the coordinator commits in Step 7.
4. **On failure:** Report which step failed and why, then STOP immediately. Do not continue.
5. The coordinator does NOT execute slash commands directly (except Steps 6 and 7). All other steps are delegated to teammates.
</rules>

## Teammate Personas

Persona files are in `references/personas/` (relative to the skill root). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Step 1: ATDD Test Architecture

**Skip if `pipeline_mode` is `tea-excluded`.**

Spawn a teammate (model: **opus**):
```
Teammate runs: /omb-tea-testarch-atdd-parallel {STORY_ID} yolo
```

## Step 2: Dev Story Implementation

Spawn a teammate (model: **sonnet**):
```
Teammate runs: /omb-dev-story-parallel {STORY_ID} yolo
```

## Step 3: Test Automation

**Skip if `pipeline_mode` is `tea-excluded`.**

Spawn a teammate (model: **sonnet**):
```
Teammate runs: /omb-tea-testarch-automate-parallel {STORY_ID} yolo
```

## Step 4: Code Review

Spawn a teammate (model: **opus**):

- **Iterations 1–2:**
  ```
  Teammate runs: /omb-code-review-parallel {STORY_ID} yolo, create action items for all the issues and classify each issue scope as MINOR, MODERATE, or SEVERE
  ```
- **Iteration 3 (final):**
  ```
  Teammate runs: /omb-code-review-parallel {STORY_ID} yolo, auto accept and fix all the issues
  ```

## Rework Decision (after Step 4)

After each code review, read the story file and check story status:

- **`done`** → proceed to Step 5
- **`in-progress`** (iterations 1–2) → determine rework scope from the highest-severity issue classification (see Rework Routing Table)
- **`in-progress`** (iteration 3, final) → proceed to Step 5 regardless — remaining issues are likely false positives

### Rework Routing Table

Use the **highest** severity among unresolved issues to determine the rework path:

| Highest Severity | Return To | Steps to Re-run |
|---|---|---|
| **MINOR** (style, naming, refactoring) | Step 2 | Dev Story → Code Review |
| **MODERATE** (logic change, API change) | Step 2 | Dev Story → Test Automation → Code Review |
| **SEVERE** (acceptance criteria were wrong) | Step 1 | ATDD → Dev Story → Test Automation → Code Review |

> When `pipeline_mode` is `tea-excluded`, all severities use the MINOR path (Dev Story → Code Review only), since ATDD and Test Automation steps are unavailable.

Maximum **3 total iterations** (initial run + up to 2 rework cycles).

## Step 5: Test Review

**Skip if `pipeline_mode` is `tea-excluded`.**

Spawn a teammate (model: **opus**):
```
Teammate runs: /omb-tea-testarch-test-review-parallel {STORY_ID} yolo
```

## Step 6: Simplify + Defer

The **coordinator directly** invokes `/simplify` (Phase 1–2 only: identify changes and launch three review agents).

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

The **coordinator directly** performs the final commit:

1. Run `git status` and `git diff --stat` to review all changes from this pipeline run.
2. Present the summary to the user and ask for confirmation.
3. On approval, stage and commit with message: `feat: implement {STORY_TITLE} (Story {STORY_ID})`

## Pipeline Summary

After all steps complete, provide a summary of the full pipeline run:

- Total iterations executed
- Rework paths taken (if any) with severity classifications
- Steps skipped (TEA-excluded steps, etc.)
- Deferred items (count and file location)
- Final story status

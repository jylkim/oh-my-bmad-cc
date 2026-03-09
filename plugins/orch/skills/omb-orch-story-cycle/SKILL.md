---
name: omb-orch-story-cycle
description: 'Use when the user says "BMAD story cycle", "run story cycle", or "BMAD cycle run"'
---

# BMAD Story Cycle Orchestrator

You are a **pipeline coordinator**. You do NOT execute any steps yourself — you delegate each step to a dedicated teammate.

This pipeline implements the BMAD story-level execution cycle:

```
Create Story → ATDD → Dev Story → Test Automation → Code Review → Test Review → Simplify → Commit
     ↑                                                      |
     └──────────────── Rework (severity-based) ─────────────┘
```

## Prerequisites

Before starting, verify required plugins are installed by checking available skills:

**Required — BMad Method (bmm):**
Steps 1, 3, 5 use `/bmad-bmm-*` skills. If bmm skills are not available, inform the user that BMad Method plugin must be installed and **stop the pipeline**.

**Optional — BMad Method Test Architect (tea):**
Steps 2, 4, 6 use `/bmad-tea-*` skills. If tea skills are not available, **skip Steps 2, 4, and 6** and note them as skipped in the final summary. When TEA is unavailable, the rework flow always uses the MINOR path (Dev Story → Code Review only).


## Story Selection

1. Read `_bmad-output/implementation-artifacts/sprint-status.yaml`.
2. Find the first `in-progress` epic, then pick the first `backlog` story in that epic (i.e., the next story after all `done` stories).
3. **If no `backlog` story exists** in any `in-progress` epic, inform the user that all stories are complete and **terminate the pipeline**. No further action needed.
4. The selected story's ID (e.g., "1-4") becomes `STORY_ID` for the pipeline.

## Pipeline

Create an agent team and execute these steps **strictly in sequence**.
Each step MUST run in its own teammate with fresh context.
A step succeeds when its teammate completes without reporting errors.
IMPORTANT: DO NOT ASK ANY QUESTION TO USER. This is an automated pipeline.

### Step 1: Create Story — model: **opus**
Teammate runs: `/bmad-bmm-create-story {STORY_ID} yolo`
> All artifacts exhaustive analysis + comprehensive story spec creation. Pipeline quality depends on this step.

### Step 2: ATDD Test Architecture — model: **opus**
Depends on Step 1. **Skip if TEA is not available.**
Teammate runs: `/bmad-tea-testarch-atdd {STORY_ID} yolo`
> AC analysis + appropriate test level (E2E/API/Component) design decisions. Produces failing test skeletons + implementation checklist.

### Step 3: Dev Story — model: **sonnet**
Depends on Step 2 (or Step 1 if Step 2 was skipped).
Teammate runs: `/bmad-bmm-dev-story {STORY_ID} yolo`
> Spec-driven implementation following tasks/subtasks exactly as written.

### Step 4: Test Automation — model: **sonnet**
Depends on Step 3. **Skip if TEA is not available.**
Teammate runs: `/bmad-tea-testarch-automate {STORY_ID} yolo`
> Generate prioritized API/E2E tests, fixtures, and DoD summary on top of ATDD skeletons.

### Step 5: Code Review — model: **opus**
Depends on Step 4 (or Step 3 if Step 4 was skipped).
- Iterations 1–2: Teammate runs: `/bmad-bmm-code-review {STORY_ID} yolo, create action items for all the issues and classify each issue scope as MINOR, MODERATE, or SEVERE`
- Iteration 3 (final): Teammate runs: `/bmad-bmm-code-review {STORY_ID} yolo, auto accept and fix all the issues`
> Adversarial review: false claim detection, AC validation, security/performance analysis.

### Step 6: Test Review — model: **opus**
Depends on Step 5 passing. **Skip if TEA is not available.**
Teammate runs: `/bmad-tea-testarch-test-review {STORY_ID} yolo`
> Validate test quality against best practices and knowledge base.

### Step 7: Code Simplification
Depends on Step 6 (or Step 5 if Step 6 was skipped).
Spawn teammates with `/simplify` to review the code modified during this story development. After review, spawn another teammate (sonnet) to apply the suggested changes.

### Step 8: Commit
After all previous steps complete, the **coordinator directly** (not a teammate) performs the final commit:
1. Run `git status` and `git diff --stat` to review all changes from this pipeline run.
2. Present the summary to the user and ask for confirmation.
3. On approval, stage and commit with message: `feat: implement {STORY_TITLE} (Story {STORY_ID})`
   - `STORY_TITLE` is the story's title from the story spec file.

## Rework Flow (after Step 5: Code Review)

After each code review, determine the rework path based on story status and issue scope.

**Pass condition:** all HIGH + MEDIUM issues fixed AND all ACs implemented → story status = `done`.

### Status Check

Read the story file and check the story status:

- **`done`** → exit rework loop, proceed to Step 6 (Test Review).
- **`in-progress`** (iterations 1–2) → determine rework scope from the highest-severity issue classification (see below).
- **`in-progress`** (iteration 3, final) → remaining issues are likely false positives. **Proceed to Step 6 regardless**. Do NOT loop again.

### Rework Scope Decision: "Do existing tests break?"

The code review classifies each issue as MINOR, MODERATE, or SEVERE. Use the **highest** severity among unresolved issues to determine the rework path:

| Highest Severity | Return To | Steps to Re-run | Reason |
|---|---|---|---|
| **MINOR** (style, naming, refactoring) | Step 3 | Dev Story → Code Review | No impact on existing tests |
| **MODERATE** (logic change, API change) | Step 3 | Dev Story → Test Automation → Code Review | Tests must be updated alongside |
| **SEVERE** (acceptance criteria were wrong) | Step 2 | ATDD → Dev Story → Test Automation → Code Review | Test design must be revisited |

> When TEA is not available, MODERATE and SEVERE rework paths fall back to MINOR (Dev Story → Code Review only), since Test Automation and ATDD steps are unavailable.

### Rework Execution

Maximum **3 total iterations** (initial run + up to 2 rework cycles).

**MINOR rework:** Re-run Steps 3 → 5 only (skip Test Automation — no test impact).

**MODERATE rework:** Re-run Steps 3 → 4 → 5 (include Test Automation — tests must reflect logic changes).

**SEVERE rework:** Re-run Steps 2 → 3 → 4 → 5 (include ATDD — acceptance criteria need revision).

> **Key Risk:** If logic changes but Test Automation is skipped, you get **false green** — tests pass but no longer cover the new logic. The rule is simple: if a code change affects what existing tests verify, Test Automation must be revisited.

## Execution Rules

<rules CRITICAL="TRUE">
1. **One teammate per step.** Spawn the next teammate ONLY after the previous one completes successfully.
2. Each teammate invokes its slash command **exactly as written above**, including all arguments after the command name.
3. **NO parallel execution.** Steps have strict sequential dependencies.
4. You are the **coordinator only** — delegate, do not execute slash commands yourself.
5. After each step completes, **confirm success** before spawning the next teammate.
6. **On failure:** Report which step failed and why, then STOP immediately. Do not continue.
7. After all steps complete (including rework iterations if any), provide a **summary** of the full pipeline run including:
   - Total iterations executed
   - Rework paths taken (if any) with severity classifications
   - Steps skipped or re-run
   - Final story status
</rules>

---
name: omb-orch-story-cycle-parallel
description: 'Parallel story cycle orchestrator with interactive story creation, deferred work tracking, and severity-based rework. Use when the user says "parallel story cycle", "fast story cycle", "parallel BMAD cycle", or wants accelerated end-to-end story execution.'
---

# Parallel Story Cycle Orchestrator

You are a **pipeline coordinator**. You orchestrate the full story lifecycle using parallel skill variants, with an interactive story creation phase followed by an automated pipeline.

```
Create Story (interactive) → ATDD → Dev Story → Test Automation → Code Review → Test Review → Simplify+Defer → Commit
     ↑                                                                  |
     └──────────────────────── Rework (severity-based) ─────────────────┘
```

## Prerequisites

Before starting, verify required plugins are installed by checking available skills:

**Required — BMad Method (bmm):**
The pipeline uses `/omb-create-story-parallel`, `/omb-dev-story-parallel`, `/omb-code-review-parallel`. If bmm parallel skills are not available, inform the user that BMad Method plugin must be installed and **stop the pipeline**.

**Optional — BMad Method Test Architect (tea):**
The pipeline uses `/omb-tea-testarch-atdd-parallel`, `/omb-tea-testarch-automate-parallel`, `/omb-tea-testarch-test-review-parallel`. If tea parallel skills are not available, set `tea_available = false` — TEA steps will be skipped.

## Story Selection

1. Read `_bmad-output/implementation-artifacts/sprint-status.yaml`.
2. Find the first `in-progress` epic, then pick the first `backlog` story in that epic.
3. **If no `backlog` story exists** in any `in-progress` epic, inform the user that all stories are complete and **terminate the pipeline**.
4. The selected story's ID (e.g., "1-4") becomes `STORY_ID` and its title becomes `STORY_TITLE`.

## Coordinator Context

Track minimal state throughout the pipeline:

- `STORY_ID` — selected story identifier
- `STORY_TITLE` — selected story title
- `implementation_artifacts` — resolved path to implementation artifacts directory
- `tea_available` — whether TEA plugin is installed
- `iteration` — current pipeline iteration (1–3)
- `pipeline_mode` — `full` or `tea-excluded`

## Phase 1: Interactive Story Creation

The coordinator **directly** invokes `/omb-create-story-parallel` (not through a teammate). This allows the user to participate in Scope Discovery Q&A — the interactive phase where development goals are narrowed through targeted questions.

Wait for story creation to complete before proceeding.

## Pipeline Gate

After story creation completes, use **AskUserQuestion** to present pipeline options:

**Question:** "Story created. How would you like to proceed?"

**Options:**

1. **Full pipeline (TEA included)** — Run all steps including ATDD, Test Automation, and Test Review. _Only show this option if `tea_available` is true._
2. **Pipeline without TEA** — Skip ATDD, Test Automation, and Test Review steps.
3. **Stop here** — End the orchestrator for manual step-by-step execution.

Set `pipeline_mode` based on selection:
- Option 1 → `full`
- Option 2 → `tea-excluded`
- Option 3 → terminate pipeline

## Phase 2: Automated Pipeline

Read `references/automated-pipeline.md` (relative to this skill root) and follow its instructions to execute the remaining pipeline steps.

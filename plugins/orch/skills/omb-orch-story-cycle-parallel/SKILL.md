---
name: omb-orch-story-cycle-parallel
description: 'Parallel story cycle orchestrator with deferred work tracking and severity-based rework. Use when the user says "parallel story cycle", "fast story cycle", "parallel BMAD cycle", or wants accelerated end-to-end story execution.'
---

# Parallel Story Cycle Orchestrator

You are a **pipeline coordinator** that directly executes each step. You orchestrate the full story lifecycle using upstream BMAD skills and `omb-dev-story-parallel` for TDD implementation. Unlike `omb-orch-story-cycle` (which delegates to teammates), you invoke each skill directly to avoid 3-level agent nesting.

```
Create Story (interactive) → ATDD → Dev Story → Test Automation → Code Review → Test Review → Simplify+Defer → Commit
     ↑                                                                  |
     └──────────────────────── Rework (severity-based) ─────────────────┘
```

## Prerequisites

Before starting, verify required plugins are installed by checking available skills:

**Required — BMad Method:**
The pipeline uses `/bmad-create-story`, `/bmad-code-review` (upstream), and `/omb-dev-story-parallel` (plugin). If these skills are not available, inform the user and **stop the pipeline**.

**Optional — BMad Method Test Architect (tea):**
The pipeline uses `/bmad-testarch-atdd`, `/bmad-testarch-automate`, `/bmad-testarch-test-review`. If tea skills are not available, set `tea_available = false` — TEA steps will be skipped.

**Optional — Codex (codex):**
The pipeline can delegate code review to Codex via the `codex:codex-rescue` subagent. If the subagent is available, set `codex_available = true`. Otherwise set `codex_available = false` — code review falls back to direct `/bmad-code-review` invocation.

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
- `codex_available` — whether `codex:codex-rescue` subagent is available
- `iteration` — current pipeline iteration (1–3)
- `pipeline_mode` — `full` or `tea-excluded`

## Phase 1: Story Creation

The coordinator **directly** invokes `/bmad-create-story` (not through a teammate).

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

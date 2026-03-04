# bmm

Parallel pipeline skills for BMAD BMM workflows. Speeds up story creation, story development, and code review by running independent tasks concurrently with Agent Teams.

## Skills

### [omb-bmm-create-story-parallel](skills/omb-bmm-create-story-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-create-story`. Overrides research steps (2–4) with 3 concurrent analysis agents.

Three research agents run simultaneously:

| Analysis | Agent | Model | Focus |
|----------|-------|-------|-------|
| Artifact Analysis | artifact-analyst | Opus | PRD, epics, and existing artifacts |
| Architecture Analysis | architecture-analyst | Sonnet | Technical architecture review |
| Tech Research | tech-researcher | Sonnet | Technology investigation |

Agents stay alive after initial research for follow-up questions during story authoring (step 5+).

### [omb-bmm-dev-story-parallel](skills/omb-bmm-dev-story-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-dev-story`. Overrides implementation steps (5–8) with a concurrent TDD pipeline.

Two pipeline modes based on context:

- **Implementation Pipeline** — Full 4-stage TDD per task, with dependency graph-based parallelism:

| Stage | Agent | Model | Focus |
|-------|-------|-------|-------|
| Red | tester-{N} | Sonnet | Write failing tests with stub signatures |
| Green | implementer-{N} | Sonnet | Minimal implementation to pass all tests |
| Refactor | refactorer-{N} | Sonnet | Improve code quality, keep tests green |
| Verify | validator-{N} | Sonnet | Full validation gates and acceptance criteria |

- **Review Pipeline** — Lightweight 2-stage fix per review item:

| Stage | Agent | Model | Focus |
|-------|-------|-------|-------|
| Fix | refactorer-{R} | Sonnet | Address review finding, keep tests green |
| Verify | validator-{R} | Sonnet | Validate fix and check for regressions |

Independent tasks/items run concurrently; dependent ones wait for upstream verification to pass.

### [omb-bmm-code-review-parallel](skills/omb-bmm-code-review-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-code-review`. Overrides the review step (3) with 4 concurrent review dimensions.

Four independent reviewers run simultaneously:

| Dimension | Agent | Model | Focus |
|-----------|-------|-------|-------|
| Git Audit | git-auditor | Sonnet | Cross-reference git changes vs story file list |
| AC Validation | ac-validator | Opus | Verify all Acceptance Criteria are met |
| Task Audit | task-auditor | Opus | Verify task completion claims |
| Code Quality | code-reviewer | Opus | Deep code quality review |

Findings are aggregated, deduplicated, and categorized by severity after all reviewers complete.

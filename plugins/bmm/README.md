# bmm

Parallel TDD pipeline skill for BMAD dev-story workflow. Upstream BMAD v6.2.2 added native subagent support to most BMM workflows — this plugin retains only `omb-dev-story-parallel`, which has no upstream equivalent.

## Skills

### [omb-dev-story-parallel](skills/omb-dev-story-parallel/SKILL.md)

Parallelized execution of `bmad-dev-story`. Overrides implementation steps (5–8) with a concurrent TDD pipeline.

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

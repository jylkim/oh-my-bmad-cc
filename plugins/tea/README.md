# tea

Pipeline skills for BMAD TEA (Test Engineering & Architecture) workflows. Parallelizes test generation and quality review with Agent Teams.

## Skills

### [omb-tea-testarch-atdd-parallel](skills/omb-tea-testarch-atdd-parallel/SKILL.md)

Parallelized execution of `bmad-tea-testarch-atdd`. Overrides step 4 (orchestration) with 2 concurrent test generation agents.

Two test generation agents run simultaneously:

| Role | Agent | Model | Focus |
|------|-------|-------|-------|
| API Tests | api-tester | Sonnet | Failing API tests from acceptance criteria (TDD red phase) |
| E2E Tests | e2e-tester | Sonnet | Failing E2E tests for user journeys (TDD red phase) |

Steps 1–3 (preflight, mode selection, test strategy) run sequentially as normal. Step 4's orchestration is replaced with Agent Team parallel dispatch. After both agents complete, aggregation (step 4c) and validation (step 5) resume sequentially.

### [omb-tea-testarch-automate-parallel](skills/omb-tea-testarch-automate-parallel/SKILL.md)

Parallelized execution of `bmad-tea-testarch-automate`. Overrides step 3 (orchestration) with up to 3 concurrent test generation agents based on `detected_stack`.

| Role | Agent | Model | Condition | Focus |
|------|-------|-------|-----------|-------|
| API Tests | api-tester | Sonnet | Always | API endpoint tests with optional Pact contracts |
| E2E Tests | e2e-tester | Sonnet | frontend/fullstack | User journey tests with resilient selectors |
| Backend Tests | backend-tester | Sonnet | backend/fullstack | Unit, integration, and contract tests |

Steps 1–2 (preflight, identify targets) run sequentially as normal. Step 3's orchestration is replaced with Agent Team parallel dispatch. After all agents complete, aggregation (step 3c) and validation (step 4) resume sequentially.

### [omb-tea-testarch-test-review-parallel](skills/omb-tea-testarch-test-review-parallel/SKILL.md)

Parallelized execution of `bmad-tea-testarch-test-review`. Overrides step 3 (orchestration) with 4 concurrent quality dimension reviewers.

| Dimension | Agent | Model | Focus |
|-----------|-------|-------|-------|
| Determinism | determinism-reviewer | Opus | Random/time dependencies, flaky patterns |
| Isolation | isolation-reviewer | Opus | Shared state, ordering dependencies, missing cleanup |
| Maintainability | maintainability-reviewer | Opus | Readability, DRY violations, structure |
| Performance | performance-reviewer | Opus | Hard sleeps, heavy setup, resource leaks |

Steps 1–2 (load context, discover tests) run sequentially as normal. Step 3's orchestration is replaced with Agent Team parallel dispatch. After all reviewers complete, score aggregation (step 3f) and report generation (step 4) resume sequentially.

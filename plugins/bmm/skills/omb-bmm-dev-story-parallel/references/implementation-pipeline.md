# Implementation Pipeline (Phase 1-5)

Full 4-stage TDD pipeline for fresh story implementation. Overrides instructions.xml steps 5-8.

## Teammate Personas

Persona files are in `personas/` (relative to this directory). Before constructing each spawn prompt, read the corresponding persona YAML and include its `persona` block as the agent's identity at the top of the prompt.

## Phase 1: Per-Task Dependency Analysis

Parse tasks from the story file and build a **per-task** execution plan:

1. **Classify every top-level task** into three categories:
   - **Implementation tasks**: Tasks that create or modify source code
   - **Test-only tasks**: Tasks solely about writing tests (e.g., "Write unit tests") — **absorbed** into implementation task pipelines
   - **Validation tasks**: Tasks for CI/verification (e.g., "Verify CI passes") — mapped to final integration-validate

2. **Absorb test-only tasks**: Examine each test-only task's subtasks and map them to the implementation task they cover. The tester agent for each implementation task will include these mapped test requirements during its Red phase.

3. **Build dependency graph** for implementation tasks:
   - For each task: identify types/modules/files it creates vs. consumes
   - Task B depends on Task A if B consumes what A creates
   - Tasks with no unresolved dependencies are immediately eligible for parallel execution

4. **Output the execution plan** before proceeding:
   ```
   Independent: [Task 1] (no dependencies) — launches immediately
   Parallel:    [Task 2, Task 4] (depend only on Task 1) — launch when Task 1 verify passes
   Sequential:  [Task 3] (depends on Task 2) — launches when Task 2 verify passes
   Absorbed:    Task 5.1 → Task 2 tester; Task 5.2 → Task 3 tester
   Integration: Task 6 (integration tests), Task 7 (CI) → integration-validate
   ```

5. If dependency analysis is inconclusive → single sequential chain (fallback)

## Phase 2: Team Setup

Create a team named `dev-{{story_key}}`.

Create tracked tasks for **EACH implementation task's** pipeline stages, with dependencies reflecting the stage order.

For each implementation task N:
- **red-{N}**: "Write failing tests for Task {N}: {title}"
- **green-{N}**: "Implement Task {N}: {title}" (blocked by red-{N})
- **refactor-{N}**: "Refactor Task {N}: {title}" (blocked by green-{N})
- **verify-{N}**: "Validate Task {N}: {title}" (blocked by refactor-{N})

For cross-task dependencies (Task N depends on Task M):
- red-{N} is blocked by verify-{M}

Final task:
- **integration-validate** (blocked by all verify-{N} tasks)

## Phase 3: Per-Task TDD Pipelines

**CRITICAL — one task, one agent, one pipeline stage:**
- NEVER assign multiple implementation tasks to a single agent
- EVERY implementation task gets its own 4-stage pipeline: Red → Green → Refactor → Verify
- Tasks with no unresolved dependencies launch CONCURRENTLY — spawn all eligible tester agents in a SINGLE message

---

### Stage A — Red: Tester Writes Failing Tests

Spawn **tester-{N}** as a background teammate (model: **sonnet**)

```
{persona from personas/tester.yaml}
You are tester-{N} of team "dev-{{story_key}}".
Your task: red-{N}

Read {installed_path}/instructions.xml and execute the following for Task {N} ONLY:
- Step 5 RED PHASE: Write FAILING tests first for task/subtask functionality
- Step 6: Author comprehensive tests (unit, integration, edge cases per story Dev Notes)

You are writing tests BEFORE implementation exists. To make tests runnable:
- Create source files with type definitions and function SIGNATURES only
- All function bodies should be stubs (no real logic) — enough for tests to compile/load but fail
- This provides enough structure for test code to reference the API surface

{base context}

Task {N} to test:
{task description with ALL subtasks and acceptance criteria}

Additional test requirements absorbed from story test tasks:
{mapped test subtasks, e.g., "From Task 5.1: registration, multi-event, lookup, exclusion, validation tests"}

When complete, mark red-{N} as completed and report to team-lead with:
- All files created/modified
- All test function names
- Stub signatures created (so implementer knows the expected API surface)
- Test runner output showing tests compile/load but fail
```

---

### Stage B — Green: Implementer Makes Tests Pass

When tester-{N} sends its completion message, **immediately** spawn **implementer-{N}** as a background teammate (model: **sonnet**)

```
{persona from personas/implementer.yaml}
You are implementer-{N} of team "dev-{{story_key}}".
Your task: green-{N}

Read {installed_path}/instructions.xml and execute the following for Task {N} ONLY:
- Step 5 GREEN PHASE: Implement MINIMAL code to make ALL pre-written tests pass

Focus on correctness only — write the simplest code that satisfies every test assertion.
Do NOT optimize, do NOT refactor, do NOT improve code structure. That is the next stage's job.

All tests for this task already exist (written in the Red phase).
Replace stubs with real implementations.
Do NOT write new tests — do NOT modify test assertions.

{base context}

Task {N} to implement:
{task description with ALL subtasks}

Pre-written test files:
{file list from tester's message}

Test functions that must pass:
{test names from tester's message}

Stub signatures to implement:
{stub list from tester's message}

When complete, mark green-{N} as completed and report to team-lead with:
- All source files created/modified
- Summary of implementation per subtask
- Test runner output showing ALL tests pass
```

---

### Stage C — Refactor: Refactorer Improves Code Quality

When implementer-{N} sends its completion message, **immediately** spawn **refactorer-{N}** as a background teammate (model: **sonnet**)

```
{persona from personas/refactorer.yaml}
You are refactorer-{N} of team "dev-{{story_key}}".
Your task: refactor-{N}

Read {installed_path}/instructions.xml and execute the following for Task {N} ONLY:
- Step 5 REFACTOR PHASE: Improve code structure while keeping ALL tests green

All tests are currently passing. Your job is to improve code quality WITHOUT breaking them.

Focus areas:
- Apply architecture patterns and coding standards from Dev Notes and project-context
- Eliminate duplication, improve naming, extract functions where clarity improves
- Ensure error handling follows project conventions
- Add rustdoc/docstrings to public API per project rules

Constraints:
- Do NOT add new functionality beyond what tests cover
- Do NOT modify test files or test assertions
- Run tests after EVERY refactoring step — if any test fails, revert that change immediately

{base context}

Task {N}:
{task description with ALL subtasks}

Source files to refactor:
{file list from implementer's message}

Test files (DO NOT MODIFY):
{file list from tester's message}

When complete, mark refactor-{N} as completed and report to team-lead with:
- All files modified (with summary of changes per file)
- Refactoring decisions made and rationale
- Test runner output confirming ALL tests still pass
```

---

### Stage D — Verify: Validator Runs Checks

When refactorer-{N} sends its completion message, **immediately** spawn **validator-{N}** as a background teammate (model: **sonnet**)

```
{persona from personas/validator.yaml}
You are validator-{N} of team "dev-{{story_key}}".
Your task: verify-{N}

Read {installed_path}/instructions.xml and execute the following for Task {N}:
- Step 7: Run validations and tests (full test suite, linting, code quality, acceptance criteria)
- Step 8 VALIDATION GATES: Verify all tests exist and pass, implementation matches spec, no regressions

Do NOT mark the task complete — report results to team-lead only.

{base context}

Task {N}:
{task description with acceptance criteria}

Changed files (test + source):
{combined file list from tester + implementer + refactorer}

Send results to team-lead:
- PASS: test counts, per-AC verification, linting/formatting status
- FAIL: FULL error output, categorized (test-failure/lint/format/regression), responsible file(s)
```

---

### Fix Loop

If validator-{N} reports FAIL:

1. Team lead determines root cause from the failure report and routes to the appropriate agent:

   | Failure type | Default fix agent | Rationale |
   |---|---|---|
   | Test failure after refactor | **refactorer-{N}** | Refactorer was last to touch code |
   | Lint/format issues | **refactorer-{N}** | Code quality is refactorer's domain |
   | Regression in other tests | **refactorer-{N}** | Side effect from refactoring |
   | Implementation fundamentally wrong (AC not met) | **implementer-{N}** | Core logic needs rework |
   | Test design error (wrong assertion/stub) | **tester-{N}** | Test itself is incorrect |

2. Send fix request to the chosen agent — include the EXACT error output from validator
3. **Routing after fix**:
   - If **refactorer-{N}** fixed → spawn new **validator-{N}** to re-verify
   - If **implementer-{N}** fixed → re-run from **refactorer-{N}** → **validator-{N}**
   - If **tester-{N}** fixed → re-run from **implementer-{N}** → **refactorer-{N}** → **validator-{N}**
4. Max 3 fix cycles per task — after that, team-lead investigates and fixes directly

---

### Per-Task Completion (Team Lead Executes Step 8)

When validator-{N} reports PASS, the team-lead **immediately**:

1. Mark task's subtasks `[x]` in the story file
2. Update File List with all files from this task's pipeline
3. Add completion note to Dev Agent Record
4. Mark verify-{N} as completed
5. Shut down all agents for this task (tester-{N}, implementer-{N}, refactorer-{N}, validator-{N})
6. If this unblocks dependent tasks → immediately launch their Red stage (eager unblocking)

## Phase 4: Integration Validation

After ALL verify-{N} tasks are completed, spawn:

**integration-validator** as a background teammate (model: **opus**)

```
{persona from personas/integration-validator.yaml}
You are integration-validator of team "dev-{{story_key}}".
Your task: integration-validate

Run the full regression test suite for the entire project.
Verify ALL existing and new tests pass together.
Check for integration issues between independently implemented tasks.
Verify ALL story-level acceptance criteria are met.

{base context}

All tasks and changed files:
{summary from all pipeline completions}

Send results to team-lead: pass/fail with full details.
```

If integration-validator reports failures:
- Identify which task caused the issue
- Send fix request to the responsible **refactorer-{N}** (default) or **implementer-{N}** (if core logic is wrong)
- After fix, re-run pipeline from the fix point through **validator-{N}**, then re-run **integration-validator**
- Max 3 cycles before manual intervention

## Phase 5: Completion & Cleanup

After integration validation passes:

1. Verify ALL task subtasks are marked `[x]` in the story file
2. Finalize File List, Dev Agent Record, and Change Log

Request all teammates to shut down gracefully.
Clean up the team and its task list.

Resume instructions.xml step 9 (story completion).

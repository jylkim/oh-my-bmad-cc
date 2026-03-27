# Changelog

## [0.3.1] - 2026-03-27

### Changed
- `omb-orch-story-cycle-parallel`: simplify rework routing — remove custom severity classification (`MINOR/MODERATE/SEVERE`), use native code review story status to determine rework
- `omb-orch-story-cycle-parallel`: remove ATDD rework path — rework always starts from Dev Story (Step 2), since `omb-dev-story-parallel` already has a red phase

### Fixed
- `omb-orch-story-cycle-parallel`: add execution rules preventing coordinator from bypassing pipeline (no direct code edits, no custom sub-agents, mandatory rework routing)

## [0.3.0] - 2026-03-27

### Changed
- `omb-orch-story-cycle-parallel`: migrate deprecated plugin skill references to upstream BMAD v6.2.2 skills
  - `/omb-create-story-parallel` → `/bmad-create-story`
  - `/omb-tea-testarch-atdd-parallel` → `/bmad-testarch-atdd`
  - `/omb-tea-testarch-automate-parallel` → `/bmad-testarch-automate`
  - `/omb-code-review-parallel` → `/bmad-code-review`
  - `/omb-tea-testarch-test-review-parallel` → `/bmad-testarch-test-review`
  - `/omb-dev-story-parallel` — retained (no upstream subagent equivalent)

## [0.2.2] - 2026-03-26

### Fixed
- `omb-orch-story-cycle`: correct upstream TEA skill name references: `bmad-tea-testarch-*` → `bmad-testarch-*` (matching BMAD v6.2.2 upstream naming)

## [0.2.1] - 2026-03-18

### Changed
- `omb-orch-story-cycle-parallel`: coordinator directly executes parallel skills instead of delegating to teammates, avoiding 3-level agent nesting

## [0.2.0] - 2026-03-18

### Added
- Parallel story cycle orchestrator (`omb-orch-story-cycle-parallel`) — uses parallel skill variants, interactive story creation with Scope Discovery, and deferred work tracking

## [0.1.2] - 2026-03-13

### Fixed
- BMM 스킬 rename에 따른 slash command 레퍼런스 업데이트

## [0.1.1] - 2026-03-09

### Fixed
- Story cycle Step 7: coordinator directly invokes `/simplify` instead of nesting through a teammate, avoiding unnecessary 3-level agent nesting

## [0.1.0] - 2026-03-09

### Added
- Story cycle orchestration skill (`omb-orch-story-cycle`) — automated 8-step story execution pipeline with severity-based rework flow

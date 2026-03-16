# Changelog

## [0.5.1] - 2026-03-16

### Changed
- Standardize all parallel skills to Phase structure with tracked tasks and aggregate dependencies
- Unify spawn prompt format, task completion marking, and heading levels across all pipeline skills

## [0.5.0] - 2026-03-16

### Added
- Next Step Handoff to pipeline skills — auto-suggests the next skill on completion via AskUserQuestion
  - `omb-create-story-parallel`: suggests ATDD (TEA) or Dev Story
  - `omb-dev-story-parallel`: suggests Test Automation (TEA) or Code Review
  - `omb-code-review-parallel`: suggests Test Review / Simplify on pass, rework path on failure (severity-based)

## [0.4.0] - 2026-03-13

### Changed
- BMM 스킬 이름에서 `bmm-` 네임스페이스 제거 (`omb-bmm-*` → `omb-*`)
- upstream 경로 `.claude/commands/` → `.claude/skills/` 마이그레이션
- XML task 경로 → workflow.md 마이그레이션

## [0.3.0] - 2026-03-13

### Added
- Context-isolated adversarial review skill for quick-spec (`omb-quick-spec-isolated-review`)

## [0.2.0] - 2026-03-09

### Added
- Context-isolated adversarial review skill for quick-dev (`omb-quick-dev-isolated-review`)

### Fixed
- Persona path resolution in all parallel pipeline skills
- Missing context propagation in all parallel pipeline skills

### Changed
- Added BMAD-style teammate personas to all parallel pipeline skills

## [0.1.0] - 2026-03-09

### Added
- Initial release with 6 parallel pipeline skills
- Create-story parallel skill (`omb-create-story-parallel`)
- Dev-story parallel skill (`omb-dev-story-parallel`)
- Code-review parallel skill (`omb-code-review-parallel`)
- Technical-research parallel skill (`omb-technical-research-parallel`)
- Market-research parallel skill (`omb-market-research-parallel`)
- Domain-research parallel skill (`omb-domain-research-parallel`)
- Plugin marketplace configuration

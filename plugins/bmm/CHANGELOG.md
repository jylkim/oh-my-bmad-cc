# Changelog

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

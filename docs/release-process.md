# Release Process

Manual version management and release process with no external tooling.

## Versioning

Follows [Semantic Versioning 2.0.0](https://semver.org/).

- Format: `MAJOR.MINOR.PATCH`
- Until `1.0.0`, the project is in initial development — MINOR bumps may include breaking changes

### Version Bump Criteria

| Change Type | Bump | Example |
|-------------|------|---------|
| Incompatible changes (skill removal, interface changes) | MAJOR | `0.x.y` → `1.0.0` |
| New skills, feature additions | MINOR | `0.1.0` → `0.2.0` |
| Bug fixes, prompt improvements, typos | PATCH | `0.1.0` → `0.1.1` |

### Where Version Is Recorded

| File | Field |
|------|-------|
| `plugins/*/.claude-plugin/plugin.json` | `"version"` |
| `plugins/*/CHANGELOG.md` | Release header |

Both must always match per plugin.

> **Note**: Claude Code uses the `version` in `plugin.json` to determine whether a plugin needs updating. Without a version bump, the user's cached plugin will not refresh.

Commits follow the [Commit Convention](commit-convention.md).

## CHANGELOG

Each plugin maintains its own `CHANGELOG.md` at `plugins/{plugin}/CHANGELOG.md`, following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format.

### Format

```markdown
# Changelog

## [0.2.0] - 2026-03-10

### Added
- Domain research parallel skill (`omb-bmm-domain-research-parallel`)

### Fixed
- Persona path resolution in all pipeline skills

### Changed
- Minimum prerequisite updated to BMAD Method v6+

## [0.1.0] - 2026-02-20

### Added
- Initial release with 6 parallel pipeline skills
- Plugin marketplace configuration
```

### Categories

| Category | Purpose |
|----------|---------|
| `Added` | New features |
| `Changed` | Changes to existing features |
| `Fixed` | Bug fixes |
| `Removed` | Removed features |
| `Deprecated` | Features marked for future removal |

Classify entries based on commit messages. Only include `docs`, `chore`, and `refactor` commits when they have user-facing impact.

## Release Checklist

Follow this sequence when releasing.

### 1. Update CHANGELOG

Add a new version section to the plugin's `CHANGELOG.md`. Review commits since the last release:

```bash
git log --oneline {last-release-commit}..HEAD -- plugins/{plugin}/
```

### 2. Update plugin.json Version

Update the `"version"` field in `plugins/{plugin}/.claude-plugin/plugin.json`.

### 3. Release Commit

```bash
git add plugins/{plugin}/CHANGELOG.md plugins/{plugin}/.claude-plugin/plugin.json
git commit -m "chore({plugin}): release v{version}"
```

### 4. Push

```bash
git push origin main
```

# Commit Convention

Follows [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

```
<type>(<scope>): <description>
```

## Types

| Type | Purpose |
|------|---------|
| `feat` | New feature (skill, persona, pipeline) |
| `fix` | Bug fix |
| `docs` | Documentation changes |
| `refactor` | Code improvement with no behavior change |
| `chore` | Build, config, and other maintenance |

## Scope

- `bmm` — Changes related to the bmm plugin
- `orch` — Changes related to the orch plugin
- Omitted — Changes unrelated to any plugin (root docs, marketplace config, etc.)

## Examples

```
feat(bmm): add domain research parallel skill
fix(bmm): correct persona path resolution in all pipeline skills
docs: add release process documentation
chore: restructure CLAUDE.md into docs/
```

## Breaking Changes

Append `!` after the type, or add a `BREAKING CHANGE:` footer in the commit body.

```
feat(bmm)!: rename pipeline phase structure

BREAKING CHANGE: Phase 3 is now "Validation" instead of "Post-processing".
Existing custom pipeline references must be updated.
```

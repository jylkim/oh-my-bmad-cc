# oh-my-bmad-cc

Scale-out extensions for [BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD). BMAD Method runs each workflow as a single agent process — this plugin scales it out to a team of specialized agents working concurrently via Claude Code Agent Teams.

## Installation

### Prerequisites

- [Claude Code](https://code.claude.com/docs/en/setup) CLI
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams) enabled (experimental feature)
- [BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD) v6+ installed in your project

#### Enabling Agent Teams

Agent Teams are experimental and disabled by default. All skills in this plugin require Agent Teams. Enable it by adding the following to your `settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

#### Installing BMAD Method

```bash
npx bmad-method install
```

Follow the installer prompts to select your AI tool (Claude Code), modules, and installation path. See the [official installation guide](https://docs.bmad-method.org/how-to/install-bmad/) for details.

### Installing the Plugin

Add this repository as a plugin marketplace, then install the plugin:

```shell
/plugin marketplace add jylkim/oh-my-bmad-cc
/plugin install bmm@oh-my-bmad
```

Restart Claude Code after installation.

## Plugins

### [bmm](plugins/bmm/README.md)

Parallel pipeline skills for BMAD BMM workflows. Parallelizes story creation, story development (TDD), code review, and research (technical, market, domain) using concurrent Agent Teams.

### [tea](plugins/tea/README.md)

Parallel pipeline skills for BMAD TEA workflows. Parallelizes test generation (ATDD, automate) and test quality review using concurrent Agent Teams.

### [orch](plugins/orch/README.md)

Pipeline orchestration skills for BMAD workflows. Automates multi-step story execution cycles by coordinating sequential workflow steps across BMM and TEA plugins.

## Roadmap

### bmm — Parallel/sub-agent skills for BMAD BMM workflows

- [x] **create-story parallel** — Parallelize artifact analysis, architecture analysis, and tech research with 3 concurrent agents.
- [x] **dev-story parallel** — TDD pipeline with per-task parallelism based on dependency graph.
- [x] **code-review parallel** — Run 4 independent review dimensions (git audit, AC validation, task audit, code quality) concurrently.
- [x] **technical-research parallel** — Run 4 research dimensions (tech stack, integration, architecture, implementation) concurrently.
- [x] **market-research parallel** — Run 4 research dimensions (behavior, pain points, decisions, competitive) concurrently.
- [x] **domain-research parallel** — Run 4 research dimensions (industry, competitive, regulatory, technical trends) concurrently.
- [x] **quick-dev isolated review** — Run adversarial review in a fresh-context sub-agent to eliminate confirmation bias.
- [x] **quick-spec isolated review** — Run adversarial review step in a fresh-context sub-agent to eliminate confirmation bias. Same pattern as quick-dev-isolated-review.
- [ ] **validate-prd parallel** — Run 10 independent validation dimensions (density, brief coverage, measurability, traceability, etc.) concurrently against the same PRD document. Same pattern as code-review-parallel.
- [ ] **check-implementation-readiness parallel** — Parallelize 3 independent validation analyses (FR coverage, UX alignment, epic quality) after requirement extraction, then aggregate into readiness report.
- [ ] **create-epics-and-stories parallel** — After epic list approval, generate stories per epic concurrently with dedicated sub-agents.
- [ ] **qa-generate-e2e-tests parallel** — Generate E2E tests per feature/epic area in parallel. Already an `autonomous: true` workflow.
- [ ] **document-project parallel** — After project type detection, parallelize per-section documentation (API, data models, UI, tests, source tree).

### tea — Parallel skills for BMAD TEA workflows

- [x] **test-review parallel** — Run 4 independent quality evaluation dimensions (determinism, isolation, maintainability, performance) concurrently, then aggregate into report.
- [x] **automate parallel** — Run 2–3 test generation workers (API, E2E, backend) concurrently based on detected stack type.
- [x] **atdd parallel** — Run 2 independent test generation workers (API tests, E2E tests) concurrently, then aggregate output.

## Development Guide

### Local Plugin Development

To test the plugin during development without permanent installation, start Claude Code with the `--plugin-dir` flag:

```bash
claude --plugin-dir /path/to/oh-my-bmad-cc
```

This loads the plugin for the current session only. Changes to skill files take effect on the next session restart.

To install permanently from a local clone:

```shell
/plugin marketplace add /path/to/oh-my-bmad-cc
/plugin install bmm@oh-my-bmad
```

To uninstall:

```shell
/plugin uninstall bmm@oh-my-bmad
```

## License

MIT

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

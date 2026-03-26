# claude-setup

Personal Claude Code plugin with reusable skills.

## Skills

- **prepare-commit** — Full commit preparation workflow: pre-flight checks (ruff, mypy, pytest), change review, and clean commit creation
- **prepare-pr** — Full PR preparation workflow: pre-flight checks, change review, PR description generation, and PR creation via gh CLI
- **copilot-check** — Review and triage GitHub Copilot review comments on a PR, then resolve threads

## Installation

1. Add the marketplace (one-time):
```
/plugin marketplace add VygantasHumble/claude-setup
```

2. Install the plugin:
```
/plugin install claude-setup@vygantas-claude-setup
```

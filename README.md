# Kimi with Superpowers

Install and configure [Superpowers](https://github.com/obra/superpowers) workflow for [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli).

## What is this?

Superpowers is a workflow system for AI coding assistants that enforces best practices like:
- Test-Driven Development (TDD)
- Systematic debugging
- Structured planning
- Subagent-driven development

This repository provides installation instructions (as an AI skill) to set up Superpowers for Kimi Code CLI, which doesn't have native Superpowers support.

## Quick Start

### Option 1: Let AI install it for you

If you're using Kimi (or any AI assistant with the Skill tool), simply invoke this skill and let the AI handle the installation:

```
Read and follow the instructions from:
https://raw.githubusercontent.com/Dqz00116/kimi-with-superpowers/main/SKILL.md
```

### Option 2: Manual installation

1. Read `SKILL.md` in this repository
2. Follow the step-by-step instructions for your platform
3. Configure your shell aliases (optional)

## Installation Overview

The installation process:

1. **Clone** the superpowers repository to `~/.kimi/superpowers/`
2. **Link** skills from superpowers to `~/.kimi/skills/`
3. **Patch** the `using-superpowers` skill for Kimi compatibility
4. **Create** a custom agent with mandatory superpowers rules
5. **Optionally patch** Kimi's built-in system.md
6. **Setup** shell aliases (optional)

## How it works

Since Kimi doesn't have a native `Skill` tool like Claude Code, this setup:

1. Places Superpowers skills in Kimi's skill discovery path (`~/.kimi/skills/`)
2. Modifies the `using-superpowers` skill to explain that Kimi should use `ReadFile` to "invoke" skills
3. Adds a "Mandatory Superpowers Invocation" section to system prompts
4. Provides a custom agent configuration that enforces these rules

When properly configured, Kimi will automatically read `using-superpowers/SKILL.md` at the start of every conversation, ensuring the Superpowers workflow is followed.

## Repository Structure

```
.
├── SKILL.md              # Main installation skill (read this!)
├── README.md             # This file
├── agents/               # Pre-configured custom agent files
│   └── superpowers/
│       ├── agent.yaml
│       ├── system.md     # System prompt with mandatory superpowers
│       ├── coder.yaml
│       ├── explore.yaml
│       └── plan.yaml
└── scripts/              # Optional helper scripts (if any)
```

## Updating

To update Superpowers to the latest version:

```bash
cd ~/.kimi/superpowers
git pull
```

Skills are linked (not copied), so they update automatically.

## Uninstallation

See the "Uninstallation" section in `SKILL.md` for detailed steps.

## Compatibility

- **Windows**: PowerShell with junction support
- **macOS**: Any modern version
- **Linux**: Any modern distribution

## Credits

- [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
- [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) by Moonshot AI

## License

MIT (same as Superpowers)

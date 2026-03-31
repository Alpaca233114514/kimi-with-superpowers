# Kimi with Superpowers

Install and configure [Superpowers](https://github.com/obra/superpowers) workflow for [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli).

## What is this?

Superpowers is a workflow system for AI coding assistants that enforces best practices like:
- Test-Driven Development (TDD)
- Systematic debugging
- Structured planning
- Subagent-driven development

This repository provides installation instructions (as an AI skill) to set up Superpowers for Kimi Code CLI, which doesn't have native Superpowers support.

## Requirements

**⚠️ IMPORTANT:** This setup requires the `feat/hook-inject-prompt` branch of Kimi Code CLI, which adds the `inject_prompt` hook feature. This feature allows automatic injection of Superpowers reminders before every conversation.

### Installing the Required Branch

```bash
# Clone the kimi-cli repository with the hook-inject-prompt feature
git clone -b feat/hook-inject-prompt https://github.com/Dqz00116/kimi-cli.git
cd kimi-cli

# Install using uv (recommended)
uv sync
uv pip install -e .

# Or run directly without installing
uv run kimi --help
```

**Windows (PowerShell):**
```powershell
# Clone the repository
git clone -b feat/hook-inject-prompt https://github.com/Dqz00116/kimi-cli.git C:\Tools\kimi-cli
cd C:\Tools\kimi-cli

# Install
uv sync
uv pip install -e .
```

## Quick Start

### Option 1: Let AI install it for you

If you're using Kimi (or any AI assistant with the Skill tool), simply invoke this skill and let the AI handle the installation:

```
Read and follow the instructions from:
https://raw.githubusercontent.com/Dqz00116/kimi-with-superpowers/main/SKILL.md
```

### Option 2: Manual installation

1. Ensure you have the `feat/hook-inject-prompt` branch of Kimi CLI installed
2. Read `SKILL.md` in this repository
3. Follow the step-by-step instructions for your platform
4. Configure your shell aliases (optional)

## Installation Overview

The installation process:

1. **Install** Kimi CLI from `feat/hook-inject-prompt` branch (see Requirements above)
2. **Clone** the superpowers repository to `~/.kimi/superpowers/`
3. **Link** skills from superpowers to `~/.kimi/skills/`
4. **Patch** the `using-superpowers` skill for Kimi compatibility
5. **Configure** `UserPromptSubmit` hook to auto-inject Superpowers reminders (NEW!)
6. **Create** a custom agent with mandatory superpowers rules (optional)
7. **Setup** shell aliases (optional)

## How it works

Since Kimi doesn't have a native `Skill` tool like Claude Code, this setup:

1. Places Superpowers skills in Kimi's skill discovery path (`~/.kimi/skills/`)
2. Modifies the `using-superpowers` skill to explain that Kimi should use `ReadFile` to "invoke" skills
3. **Uses the new `inject_prompt` hook feature** to automatically inject Superpowers reminders before every conversation
4. Provides a custom agent configuration that enforces these rules

The key innovation is the `UserPromptSubmit` hook with `inject_prompt`, which:
- Automatically injects the Superpowers invocation instruction as a system reminder
- Works with EVERY conversation without requiring the `--agent` flag
- Eliminates the need to patch Kimi's built-in system.md

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
├── prompts/              # Prompt templates for hook injection
│   └── superpowers-reminder.md
└── scripts/              # Optional helper scripts (if any)
```

## Updating

To update Superpowers to the latest version:

```bash
cd ~/.kimi/superpowers
git pull
```

Skills are linked (not copied), so they update automatically.

To update Kimi CLI to the latest `feat/hook-inject-prompt` branch:

```bash
cd <path-to-kimi-cli>
git pull origin feat/hook-inject-prompt
uv sync
```

## Uninstallation

See the "Uninstallation" section in `SKILL.md` for detailed steps.

## Compatibility

- **Kimi CLI**: Requires `feat/hook-inject-prompt` branch
- **Windows**: PowerShell with junction support
- **macOS**: Any modern version
- **Linux**: Any modern distribution

## Credits

- [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
- [Kimi Code CLI](https://github.com/MoonshotAI/kimi-cli) by Moonshot AI
- `feat/hook-inject-prompt` branch by [Dqz00116](https://github.com/Dqz00116)

## License

MIT (same as Superpowers)

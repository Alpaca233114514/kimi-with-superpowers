---
name: install-superpowers-for-kimi
description: Install and configure Superpowers workflow for Kimi Code CLI on any platform (Windows/macOS/Linux)
---

# Install Superpowers for Kimi Code CLI

This skill guides you through installing the Superpowers workflow system for Kimi Code CLI.

## Prerequisites

- Kimi Code CLI installed and working
- Git installed
- Python 3.8+ installed (for running helper scripts if needed)

## Installation Steps

### Step 1: Clone Superpowers Repository

Clone the superpowers repository to your local Kimi configuration directory:

```bash
# Create .kimi directory if it doesn't exist
mkdir -p ~/.kimi

# Clone superpowers repository
git clone --depth 1 https://github.com/obra/superpowers.git ~/.kimi/superpowers
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.kimi"
git clone --depth 1 https://github.com/obra/superpowers.git "$env:USERPROFILE\.kimi\superpowers"
```

### Step 2: Create Skill Links

Superpowers skills need to be linked from `~/.kimi/superpowers/skills/` to `~/.kimi/skills/` so Kimi can discover them.

**On macOS/Linux:**
```bash
mkdir -p ~/.kimi/skills

# Create symlinks for all superpowers skills
for skill in brainstorming dispatching-parallel-agents executing-plans \
    finishing-a-development-branch receiving-code-review requesting-code-review \
    subagent-driven-development systematic-debugging test-driven-development \
    using-git-worktrees using-superpowers verification-before-completion \
    writing-plans writing-skills; do
    ln -sf ~/.kimi/superpowers/skills/$skill ~/.kimi/skills/$skill
done
```

**On Windows (PowerShell as Administrator):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.kimi\skills"

$skills = @(
    "brainstorming", "dispatching-parallel-agents", "executing-plans",
    "finishing-a-development-branch", "receiving-code-review", "requesting-code-review",
    "subagent-driven-development", "systematic-debugging", "test-driven-development",
    "using-git-worktrees", "using-superpowers", "verification-before-completion",
    "writing-plans", "writing-skills"
)

foreach ($skill in $skills) {
    $target = "$env:USERPROFILE\.kimi\skills\$skill"
    $source = "$env:USERPROFILE\.kimi\superpowers\skills\$skill"
    if (Test-Path $target) { Remove-Item $target -Recurse -Force }
    cmd /c mklink /J "$target" "$source"
}
```

### Step 3: Patch using-superpowers Skill for Kimi

Kimi doesn't have a native `Skill` tool like Claude Code. We need to modify the `using-superpowers` skill to explain how to invoke skills via `ReadFile`.

Edit `~/.kimi/superpowers/skills/using-superpowers/SKILL.md`:

1. Change the description to:
   ```yaml
   description: MANDATORY — Read this skill FIRST at the start of EVERY conversation before any response or action
   ```

2. In the "How to Access Skills" section, add Kimi-specific instructions after "In Gemini CLI:":
   ```markdown
   **In Kimi Code CLI:** Kimi does not have a native `Skill` tool. To invoke a skill, you MUST use the `ReadFile` tool to read the skill's `SKILL.md` file from its path (listed in the system prompt under Available skills). After reading the skill content, follow its instructions exactly. This is how you "invoke" a skill in Kimi.
   ```

3. In "The Rule" section, add after the first paragraph:
   ```markdown
   **In Kimi Code CLI specifically:** Before responding to the user's first message in any conversation, you MUST use `ReadFile` to read `using-superpowers/SKILL.md` (from the skill paths listed in the system prompt). Then, if any other skill might apply, use `ReadFile` to read that skill's `SKILL.md` as well. Do not skip this step.
   ```

### Step 4: Create Custom Superpowers Agent

Create a custom agent configuration that includes mandatory superpowers invocation rules.

**Create directory structure:**
```bash
mkdir -p ~/.kimi/agents/superpowers
```

**Create `~/.kimi/agents/superpowers/agent.yaml`:**
```yaml
version: 1
agent:
  name: "superpowers"
  extend: default
  system_prompt_path: ./system.md
  system_prompt_args:
    ROLE_ADDITIONAL: ""
```

**Create `~/.kimi/agents/superpowers/system.md`:**

Copy the content from Kimi's default system.md (located at `<kimi_cli_path>/agents/default/system.md`), then add this section after "## How to use skills":

```markdown
## Mandatory Superpowers Invocation

**CRITICAL RULE:** At the start of EVERY conversation, before any response, action, or clarifying question, you MUST invoke the `using-superpowers` skill. In Kimi Code CLI, this means you MUST use the `ReadFile` tool to read the `SKILL.md` file of the `using-superpowers` skill (path listed under Available skills above).

After reading `using-superpowers`, determine if any other skills apply to the current task. If there is even a 1% chance another skill might apply, you MUST read that skill's `SKILL.md` as well before proceeding.

Do not skip this step. Do not respond to the user before reading `using-superpowers`.
```

**Also copy supporting files:**
```bash
# Find Kimi CLI installation path and copy supporting agent files
KIMI_AGENTS_PATH="<path_to_kimi_cli>/agents/default"
cp "$KIMI_AGENTS_PATH/coder.yaml" ~/.kimi/agents/superpowers/
cp "$KIMI_AGENTS_PATH/explore.yaml" ~/.kimi/agents/superpowers/
cp "$KIMI_AGENTS_PATH/plan.yaml" ~/.kimi/agents/superpowers/
```

### Step 5: Optionally Patch Kimi's Built-in system.md

To ensure superpowers is used even without the custom agent, patch Kimi's built-in system.md:

1. Find Kimi CLI installation:
   - Windows: Usually in `%APPDATA%/uv/tools/kimi-cli/Lib/site-packages/kimi_cli/` or `%APPDATA%/pip/...`
   - macOS/Linux: Usually in `~/.local/lib/python3.x/site-packages/kimi_cli/` or `/usr/local/lib/python3.x/site-packages/kimi_cli/`

2. Edit `<kimi_cli_path>/agents/default/system.md` and add the same "Mandatory Superpowers Invocation" section as in Step 4.

### Step 6: Setup Shell Aliases (Optional but Recommended)

To use the superpowers agent by default, add aliases to your shell.

**Bash (~/.bashrc):**
```bash
alias kimi='kimi --agent-file "$HOME/.kimi/agents/superpowers/agent.yaml"'
alias kimi-cli='kimi-cli --agent-file "$HOME/.kimi/agents/superpowers/agent.yaml"'
```

**Zsh (~/.zshrc):**
```zsh
alias kimi='kimi --agent-file "$HOME/.kimi/agents/superpowers/agent.yaml"'
alias kimi-cli='kimi-cli --agent-file "$HOME/.kimi/agents/superpowers/agent.yaml"'
```

**Fish (~/.config/fish/config.fish):**
```fish
function kimi
    command kimi --agent-file "$HOME/.kimi/agents/superpowers/agent.yaml" $argv
end

function kimi-cli
    command kimi-cli --agent-file "$HOME/.kimi/agents/superpowers/agent.yaml" $argv
end
```

**PowerShell ($PROFILE):**
```powershell
function kimi {
    & "$env:USERPROFILE\.local\bin\kimi.exe" --agent-file "$env:USERPROFILE\.kimi\agents\superpowers\agent.yaml" @args
}

function kimi-cli {
    & "$env:USERPROFILE\.local\bin\kimi-cli.exe" --agent-file "$env:USERPROFILE\.kimi\agents\superpowers\agent.yaml" @args
}
```

## Verification

1. Start a new shell session or source your profile
2. Run `kimi --help` to verify it works
3. Start a conversation - the AI should automatically read `using-superpowers/SKILL.md` before responding

## Updating Superpowers

To update to the latest version:

```bash
cd ~/.kimi/superpowers
git pull
```

Skills are linked via symlinks/junctions, so they update automatically.

## Uninstallation

1. Remove skill links:
   ```bash
   rm -rf ~/.kimi/skills/*
   ```

2. Remove superpowers repository:
   ```bash
   rm -rf ~/.kimi/superpowers
   ```

3. Remove custom agent:
   ```bash
   rm -rf ~/.kimi/agents/superpowers
   ```

4. Remove shell aliases from your profile files

5. Restore original system.md if you patched it (from the `.backup` file created during patching)

## Troubleshooting

**Skills not showing up:**
- Check that symlinks/junctions were created correctly: `ls -la ~/.kimi/skills/`
- Verify the superpowers repo was cloned: `ls ~/.kimi/superpowers/skills/`

**Permission denied on Windows:**
- Creating junctions requires Administrator privileges on Windows. Run PowerShell as Administrator.

**Kimi can't find skills:**
- Kimi looks for skills in `~/.kimi/skills/` by default
- Make sure the directory exists and contains the skill links

**Agent file not found:**
- Use absolute paths in shell aliases
- Verify the agent.yaml exists: `cat ~/.kimi/agents/superpowers/agent.yaml`

## Platform-Specific Notes

### Windows
- Use junctions (`mklink /J`) instead of symlinks for better compatibility
- PowerShell execution policy may need to be set: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

### macOS
- No special requirements

### Linux
- No special requirements

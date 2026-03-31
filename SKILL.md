---
name: install-superpowers-for-kimi
description: Install and configure Superpowers workflow for Kimi Code CLI (feat/hook-inject-prompt branch) on any platform (Windows/macOS/Linux)
---

# Install Superpowers for Kimi Code CLI

This skill guides you through installing the Superpowers workflow system for Kimi Code CLI with automatic skill invocation via the `inject_prompt` hook feature.

## ⚠️ Prerequisites - IMPORTANT

**This installation requires the `feat/hook-inject-prompt` branch of Kimi Code CLI**, which adds support for the `inject_prompt` hook feature. This feature is essential for automatically injecting Superpowers reminders before every conversation.

### Step 0: Install Kimi CLI from feat/hook-inject-prompt Branch

**Option A: Clone and Install (Recommended)**

```bash
# macOS/Linux
git clone -b feat/hook-inject-prompt https://github.com/Dqz00116/kimi-cli.git ~/kimi-cli
cd ~/kimi-cli
uv sync
uv pip install -e .

# Verify installation
kimi --version
```

**Windows (PowerShell):**
```powershell
# Clone to a permanent location
git clone -b feat/hook-inject-prompt https://github.com/Dqz00116/kimi-cli.git "$env:USERPROFILE\kimi-cli"
cd "$env:USERPROFILE\kimi-cli"

# Install dependencies and package
uv sync
uv pip install -e .

# Add to PATH (optional - create wrapper scripts)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Tools"

@'
@echo off
"%USERPROFILE%\kimi-cli\.venv\Scripts\kimi.exe" %*
'@ | Set-Content "$env:USERPROFILE\Tools\kimi.cmd"

@'
@echo off
"%USERPROFILE%\kimi-cli\.venv\Scripts\kimi.exe" %*
'@ | Set-Content "$env:USERPROFILE\Tools\kimi-cli.cmd"

# Add to user PATH
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";" + "$env:USERPROFILE\Tools", "User")
```

**Option B: Run Without Installing**

If you prefer not to install, you can run directly from the source:

```bash
cd ~/kimi-cli
uv run kimi --help
```

### Verify Hook Support

After installation, verify that the `inject_prompt` feature is available:

```bash
# Should show UserPromptSubmit in the list of supported events
kimi --help | grep -A 5 "hooks"
```

---

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

### Step 4: Configure UserPromptSubmit Hook (NEW - REQUIRES feat/hook-inject-prompt BRANCH)

This is the **key step** that enables automatic Superpowers invocation. The `inject_prompt` hook feature from the `feat/hook-inject-prompt` branch allows us to automatically inject a reminder before every conversation.

**Create the prompts directory and reminder file:**

```bash
mkdir -p ~/.kimi/prompts

# Create the Superpowers reminder prompt
cat > ~/.kimi/prompts/superpowers-reminder.md << 'EOF'
[SYSTEM REMINDER - MANDATORY]

Before responding to the user's message, you MUST invoke the Superpowers workflow by reading the using-superpowers skill.

**REQUIRED ACTION:**
1. Use ReadFile to read: ~/.kimi/skills/using-superpowers/SKILL.md
2. Follow the instructions in that skill EXACTLY
3. If any other skill might apply (even 1% chance), read that skill's SKILL.md as well

**Available Superpowers Skills:**
- brainstorming - For open-ended exploration and idea generation
- dispatching-parallel-agents - For running multiple subagents in parallel
- executing-plans - For executing pre-written implementation plans
- finishing-a-development-branch - For wrapping up feature branches
- receiving-code-review - For processing code review feedback
- requesting-code-review - For requesting human code review
- subagent-driven-development - For complex multi-file changes using subagents
- systematic-debugging - For methodical debugging of issues
- test-driven-development - For TDD workflow (write tests first)
- using-git-worktrees - For managing multiple git worktrees
- using-superpowers - MANDATORY - Read this FIRST before every response
- verification-before-completion - For verifying work before finishing
- writing-plans - For creating implementation plans
- writing-skills - For creating new skills

Do NOT skip this step. Do NOT respond before reading using-superpowers.
EOF
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.kimi\prompts"

$promptContent = @"
[SYSTEM REMINDER - MANDATORY]

Before responding to the user's message, you MUST invoke the Superpowers workflow by reading the using-superpowers skill.

**REQUIRED ACTION:**
1. Use ReadFile to read: ~/.kimi/skills/using-superpowers/SKILL.md
2. Follow the instructions in that skill EXACTLY
3. If any other skill might apply (even 1% chance), read that skill's SKILL.md as well

**Available Superpowers Skills:**
- brainstorming - For open-ended exploration and idea generation
- dispatching-parallel-agents - For running multiple subagents in parallel
- executing-plans - For executing pre-written implementation plans
- finishing-a-development-branch - For wrapping up feature branches
- receiving-code-review - For processing code review feedback
- requesting-code-review - For requesting human code review
- subagent-driven-development - For complex multi-file changes using subagents
- systematic-debugging - For methodical debugging of issues
- test-driven-development - For TDD workflow (write tests first)
- using-git-worktrees - For managing multiple git worktrees
- using-superpowers - MANDATORY - Read this FIRST before every response
- verification-before-completion - For verifying work before finishing
- writing-plans - For creating implementation plans
- writing-skills - For creating new skills

Do NOT skip this step. Do NOT respond before reading using-superpowers.
"@

$promptContent | Set-Content "$env:USERPROFILE\.kimi\prompts\superpowers-reminder.md" -Encoding UTF8
```

**Add the hook to your config.toml:**

Edit `~/.kimi/config.toml` and add:

```toml
[[hooks]]
event = "UserPromptSubmit"
inject_prompt = "~/.kimi/prompts/superpowers-reminder.md"
timeout = 5
```

**Windows:** The path `~/.kimi/prompts/superpowers-reminder.md` will be automatically expanded to your user profile directory.

### Step 5: Create Custom Superpowers Agent (Optional)

If you want to use a custom agent configuration with additional Superpowers rules:

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

### Step 6: Setup Shell Aliases (Optional but Recommended)

To use the superpowers agent by default, add aliases to your shell.

**Note:** With the `inject_prompt` hook configured (Step 4), the custom agent is optional. The hook will inject Superpowers reminders regardless of which agent you use.

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
    & "$env:USERPROFILE\kimi-cli\.venv\Scripts\kimi.exe" --agent-file "$env:USERPROFILE\.kimi\agents\superpowers\agent.yaml" @args
}

function kimi-cli {
    & "$env:USERPROFILE\kimi-cli\.venv\Scripts\kimi.exe" --agent-file "$env:USERPROFILE\.kimi\agents\superpowers\agent.yaml" @args
}

# Optional: Simple alias without custom agent (hook will still inject Superpowers)
function k {
    & "$env:USERPROFILE\kimi-cli\.venv\Scripts\kimi.exe" @args
}
```

---

## Verification

1. **Verify Kimi CLI is installed correctly:**
   ```bash
   kimi --version
   ```

2. **Verify the hook is configured:**
   ```bash
   # In interactive mode, type:
   /hooks
   ```
   You should see `UserPromptSubmit: 1 hook(s)`

3. **Start a conversation:**
   ```bash
   kimi
   ```
   
   The AI should automatically read `using-superpowers/SKILL.md` before responding to your first message. You should see a tool call like:
   ```
   ReadFile(path="~/.kimi/skills/using-superpowers/SKILL.md")
   ```

4. **Test without custom agent:**
   ```bash
   # Even without --agent flag, the hook should still inject Superpowers
   kimi
   ```

---

## Updating

### Update Superpowers

```bash
cd ~/.kimi/superpowers
git pull
```

Skills are linked via symlinks/junctions, so they update automatically.

### Update Kimi CLI (feat/hook-inject-prompt branch)

```bash
cd <path-to-your-kimi-cli>
git pull origin feat/hook-inject-prompt
uv sync
```

---

## Uninstallation

1. **Remove hook from config:**
   Edit `~/.kimi/config.toml` and remove the `[[hooks]]` section for `UserPromptSubmit`.

2. **Remove skill links:**
   ```bash
   rm -rf ~/.kimi/skills/*
   ```

3. **Remove superpowers repository:**
   ```bash
   rm -rf ~/.kimi/superpowers
   ```

4. **Remove custom agent:**
   ```bash
   rm -rf ~/.kimi/agents/superpowers
   ```

5. **Remove prompt file:**
   ```bash
   rm -rf ~/.kimi/prompts
   ```

6. **Remove shell aliases** from your profile files

7. **(Optional) Remove Kimi CLI:**
   ```bash
   cd <path-to-your-kimi-cli>
   uv pip uninstall kimi-cli
   ```

---

## Troubleshooting

**"inject_prompt not supported" or hook not working:**
- You are NOT using the `feat/hook-inject-prompt` branch. Please follow Step 0 to install the correct branch.

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

**Hook not triggering:**
- Verify config.toml syntax: `cat ~/.kimi/config.toml`
- Check that the prompt file exists: `cat ~/.kimi/prompts/superpowers-reminder.md`
- Try using an absolute path instead of `~` in the inject_prompt field

---

## Platform-Specific Notes

### Windows
- Use junctions (`mklink /J`) instead of symlinks for better compatibility
- PowerShell execution policy may need to be set: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
- The `~` in paths is automatically expanded to `%USERPROFILE%`

### macOS
- No special requirements

### Linux
- No special requirements

---

## How It Works (Technical Details)

The `feat/hook-inject-prompt` branch adds the `inject_prompt` field to the HookDef configuration. When a `UserPromptSubmit` hook has `inject_prompt` set:

1. The hook engine reads the file content (or uses the literal string)
2. The content is injected as `additional_context` in the HookResult
3. KimiSoul receives this context and appends it as a system reminder message
4. The LLM sees this reminder before processing the user's input
5. The reminder instructs the LLM to read the Superpowers skill

This approach is superior to the old method because:
- ✅ Works with ANY agent (no need for `--agent superpowers`)
- ✅ No need to patch Kimi's built-in system.md
- ✅ Can be toggled on/off by editing config.toml
- ✅ Multiple hooks can inject different contexts

---

## Migration from Old Method

If you previously installed Superpowers using the old method (patching system.md):

1. Install the `feat/hook-inject-prompt` branch (Step 0)
2. Follow Step 4 to configure the `inject_prompt` hook
3. Remove the old "Mandatory Superpowers Invocation" section from:
   - `~/.kimi/agents/superpowers/system.md` (if using custom agent)
   - Kimi's built-in system.md (if you patched it)
4. The hook-based method will now handle Superpowers invocation automatically

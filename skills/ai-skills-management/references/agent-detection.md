# Agent Detection Reference

Complete detection matrix for AI tools, agents, and configurations.

---

## Project-Level Detection

Scan the project root for these files and folders:

### GitHub Copilot / VS Code

| Path                                     | What it indicates                 |
| ---------------------------------------- | --------------------------------- |
| `.github/copilot-instructions.md`        | Copilot custom instructions       |
| `.github/agents/*.agent.md`              | Custom Copilot agents             |
| `.github/skills/*/SKILL.md`              | Copilot skills installed          |
| `.github/prompts/*.prompt.md`            | Copilot reusable prompts          |
| `.github/instructions/*.instructions.md` | Copilot file instructions         |
| `.vscode/settings.json`                  | Check for `github.copilot.*` keys |
| `.vscode/mcp.json`                       | MCP servers for VS Code           |

### Claude Code

| Path                          | What it indicates            |
| ----------------------------- | ---------------------------- |
| `.claude/settings.json`       | Claude Code project settings |
| `.claude/settings.local.json` | Claude Code local settings   |
| `.claude/commands/*.md`       | Claude slash commands        |
| `.claude/skills/*/SKILL.md`   | Claude skills installed      |
| `CLAUDE.md`                   | Claude context file (root)   |
| `.claude/CLAUDE.md`           | Claude context file (hidden) |

### Codex (OpenAI)

| Path                       | What it indicates                   |
| -------------------------- | ----------------------------------- |
| `.codex/`                  | Codex configuration folder          |
| `.codex/instructions.md`   | Codex custom instructions           |
| `.codex/skills/*/SKILL.md` | Codex skills installed              |
| `AGENTS.md`                | Shared context (also used by Codex) |

### Gemini CLI

| Path                        | What it indicates        |
| --------------------------- | ------------------------ |
| `.gemini/`                  | Gemini CLI configuration |
| `.gemini/settings.json`     | Gemini settings          |
| `.gemini/skills/*/SKILL.md` | Gemini skills installed  |
| `GEMINI.md`                 | Gemini context file      |

### OpenCode

| Path                          | What it indicates         |
| ----------------------------- | ------------------------- |
| `.opencode/`                  | OpenCode configuration    |
| `.opencode/agents/*.md`       | OpenCode agents           |
| `.opencode/skills/*/SKILL.md` | OpenCode skills installed |
| `opencode.json`               | OpenCode project config   |

### Continue

| Path                    | What it indicates        |
| ----------------------- | ------------------------ |
| `.continue/`            | Continue configuration   |
| `.continue/config.json` | Continue settings        |
| `.continue/config.yaml` | Continue settings (YAML) |
| `.continuerules`        | Continue rules file      |

### Cline / Roo

| Path          | What it indicates |
| ------------- | ----------------- |
| `.clinerules` | Cline rules file  |
| `.roo/`       | Roo configuration |
| `.roo/rules/` | Roo rules         |
| `.roomodes`   | Roo custom modes  |

### Aider

| Path                        | What it indicates     |
| --------------------------- | --------------------- |
| `.aider.conf.yml`           | Aider configuration   |
| `.aiderignore`              | Aider ignore patterns |
| `.aider.model.settings.yml` | Aider model config    |

### Shared / Generic

| Path                     | What it indicates        |
| ------------------------ | ------------------------ |
| `AGENTS.md`              | Shared AI context file   |
| `CONVENTIONS.md`         | Project conventions      |
| `.mcp.json` / `mcp.json` | MCP server configuration |
| `.cursorrules`           | Cursor AI rules          |
| `.windsurfrules`         | Windsurf rules           |

---

## System-Level Detection

### CLI Tools

Run these commands to detect installed AI CLIs. Use the appropriate error-suppression for the OS.

| Tool        | Detection Command (Windows)  | Detection Command (Unix)         |
| ----------- | ---------------------------- | -------------------------------- |
| Claude Code | `claude --version 2>$null`   | `claude --version 2>/dev/null`   |
| Codex       | `codex --version 2>$null`    | `codex --version 2>/dev/null`    |
| Gemini CLI  | `gemini --version 2>$null`   | `gemini --version 2>/dev/null`   |
| OpenCode    | `opencode --version 2>$null` | `opencode --version 2>/dev/null` |
| Aider       | `aider --version 2>$null`    | `aider --version 2>/dev/null`    |
| Continue    | `continue --version 2>$null` | `continue --version 2>/dev/null` |

If the command returns a version string (exit code 0), the tool is installed.

### VS Code Extensions

Run `code --list-extensions` and filter for AI-related extensions:

| Extension ID                          | Tool                |
| ------------------------------------- | ------------------- |
| `GitHub.copilot`                      | GitHub Copilot      |
| `GitHub.copilot-chat`                 | GitHub Copilot Chat |
| `Continue.continue`                   | Continue            |
| `saoudrizwan.claude-dev`              | Cline               |
| `rooveterinaryinc.roo-cline`          | Roo                 |
| `GoogleCloudTools.gemini-code-assist` | Gemini Code Assist  |
| `sourcegraph.cody-ai`                 | Sourcegraph Cody    |
| `Cursor.*`                            | Cursor-related      |
| `TabNine.tabnine-vscode`              | Tabnine             |
| `AmazonWebServices.amazon-q-vscode`   | Amazon Q            |

### Global Configuration Folders

| OS        | Tool                    | Global Path                       |
| --------- | ----------------------- | --------------------------------- |
| Windows   | Claude Code             | `%USERPROFILE%\.claude\`          |
| Windows   | Copilot (VS Code)       | `%APPDATA%\Code\User\prompts\`    |
| Windows   | Copilot (skills)        | `%USERPROFILE%\.copilot\skills\`  |
| Windows   | Copilot (agents skills) | `%USERPROFILE%\.agents\skills\`   |
| Windows   | OpenCode                | `%USERPROFILE%\.config\opencode\` |
| Windows   | Continue                | `%USERPROFILE%\.continue\`        |
| Windows   | Gemini                  | `%APPDATA%\gemini\`               |
| Linux/Mac | Claude Code             | `~/.claude/`                      |
| Linux/Mac | Copilot (VS Code)       | `~/.config/Code/User/prompts/`    |
| Linux/Mac | Copilot (skills)        | `~/.copilot/skills/`              |
| Linux/Mac | Copilot (agents skills) | `~/.agents/skills/`               |
| Linux/Mac | OpenCode                | `~/.config/opencode/`             |
| Linux/Mac | Continue                | `~/.continue/`                    |
| Linux/Mac | Gemini                  | `~/.config/gemini/`               |

---

## Output Format

Present detections progressively. Group by category:

```
Scanning AI environment...

  Project configuration:
    ✓ .github/ (Copilot config found)
    ✓ .claude/ (Claude Code config found)
    ✓ AGENTS.md (shared context)
    ✗ .codex/ (not found)
    ✗ .gemini/ (not found)
    ✗ .opencode/ (not found)

  System tools:
    ✓ Claude Code CLI (v1.x.x)
    ✓ VS Code with Copilot extension
    ✗ Codex CLI (not installed)
    ✗ Gemini CLI (not installed)
    ✓ OpenCode CLI (v0.x.x)

  Summary: 2 agents configured, 3 tools detected
```

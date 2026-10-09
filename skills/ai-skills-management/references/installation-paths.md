# Installation Paths Reference

Complete path matrix for linking skills, agents, prompts, and instructions per AI agent and OS.

---

## Destination Paths by Agent

### GitHub Copilot (VS Code)

| Artifact Type | Project Path                                  | Notes                                   |
| ------------- | --------------------------------------------- | --------------------------------------- |
| Skill         | `.github/skills/<skill-name>/`                | Folder containing `SKILL.md`            |
| Agent         | `.github/agents/<name>.agent.md`              | Copilot format; convert if needed       |
| Prompt        | `.github/prompts/<name>.prompt.md`            | Single file                             |
| Instruction   | `.github/instructions/<name>.instructions.md` | Single file                             |
| Context       | `.github/copilot-instructions.md`             | Do NOT overwrite; append or skip        |

VS Code Copilot also reads Claude-format agents from `.claude/agents/`. If both Copilot
and Claude Code are targets, ask whether one `.claude/agents/` file is enough for both.

### Claude Code

| Artifact Type  | Project Path                   | Notes                                        |
| -------------- | ------------------------------ | -------------------------------------------- |
| Skill          | `.claude/skills/<skill-name>/` | Folder containing `SKILL.md`                 |
| Agent          | `.claude/agents/<name>.md`     | Subagent, Claude format; convert if needed   |
| Prompt         | `.claude/commands/<name>.md`   | Slash command; single markdown file          |
| Context        | `CLAUDE.md` (root)             | Do NOT overwrite; append or skip             |
| Settings       | `.claude/settings.json`        | Do NOT overwrite                             |

> Agents are **never** installed in `.claude/commands/`. A command is a prompt the user
> invokes with `/name`; an agent is a subagent Claude delegates to. They are different
> artifacts.

### Codex (OpenAI)

| Artifact Type | Project Path                   | Notes                                         |
| ------------- | ------------------------------ | --------------------------------------------- |
| Skill         | `.agents/skills/<skill-name>/` | Folder containing `SKILL.md`                  |
| Agent         | `.codex/agents/<name>.toml`    | TOML; always a converted copy                 |
| Prompt        | —                              | No project-level prompts; offer it as a skill |
| Context       | `AGENTS.md` (root)             | Do NOT overwrite                              |
| Settings      | `.codex/config.toml`           | Do NOT overwrite                              |

### Gemini CLI

| Artifact Type | Project Path                                        | Notes                                    |
| ------------- | --------------------------------------------------- | ---------------------------------------- |
| Skill         | `.gemini/skills/<skill-name>/` (or `.agents/skills/`) | Folder containing `SKILL.md`           |
| Agent         | `.gemini/agents/<name>.md`                          | Gemini format; convert if needed         |
| Prompt        | `.gemini/commands/<name>.toml`                      | TOML: `description` + `prompt = '''…'''` |
| Context       | `GEMINI.md` (root)                                  | Do NOT overwrite                         |
| Settings      | `.gemini/settings.json`                             | Do NOT overwrite                         |

For Gemini prompts, put the `.prompt.md` body in `prompt` and replace `${input:…}` /
`$ARGUMENTS` placeholders with `{{args}}`.

### OpenCode

| Artifact Type | Project Path                     | Notes                                       |
| ------------- | -------------------------------- | ------------------------------------------- |
| Skill         | `.opencode/skills/<skill-name>/` | Folder containing `SKILL.md`                |
| Agent         | `.opencode/agents/<name>.md`     | OpenCode format (`mode: subagent`); convert |
| Prompt        | `.opencode/commands/<name>.md`   | `description` frontmatter + body            |
| Context       | `AGENTS.md` / `opencode.json`    | Do NOT overwrite                            |

For converting agents between formats, see [agent-formats.md](agent-formats.md).

---

## Shared Skill Folders (Avoid Duplicates)

Several tools read skills from more than one folder:

| Folder            | Read by                                 |
| ----------------- | --------------------------------------- |
| `.claude/skills/` | Claude Code, OpenCode, VS Code Copilot  |
| `.agents/skills/` | Codex, Gemini CLI, OpenCode             |
| `.github/skills/` | GitHub Copilot                          |
| `.gemini/skills/` | Gemini CLI                              |
| `.opencode/skills/` | OpenCode                              |

Before linking a skill for a target, check whether the target already discovers it from
another installed folder. If it does, **skip** that link and say so — duplicate skill
names make tools pick one copy or show it twice. Examples:

- Claude Code + OpenCode → link in `.claude/skills/` only.
- Codex + Gemini CLI → link in `.agents/skills/` only.
- Claude Code + Codex + Gemini → `.claude/skills/` + `.agents/skills/`.

If a duplicate cannot be avoided (OpenCode reads both `.claude/skills/` and
`.agents/skills/`), both links point to the same source, so the content is identical.
Tell the user about it instead of failing.

---

## Link Commands by OS

### Windows — Junction (Primary)

```powershell
# Create junction (no admin required, works for directories)
cmd /c mklink /J "<destination>" "<source>"

# Example:
cmd /c mklink /J ".github\skills\component-patterns" "C:\AI-Skills\react\component-patterns"
```

**Verification:**

```powershell
# Check if a path is a junction
fsutil reparsepoint query "<path>"
# Or: (Get-Item "<path>").Attributes -match "ReparsePoint"
```

**Removal (safe — does NOT delete source):**

```powershell
# Remove junction only (source folder untouched)
cmd /c rmdir "<junction-path>"
```

### Windows — Symlink (Secondary)

```powershell
# Requires Developer Mode enabled OR elevated prompt
cmd /c mklink /D "<destination>" "<source>"
```

**Verification:**

```powershell
# Same as junction
fsutil reparsepoint query "<path>"
```

**Removal:**

```powershell
cmd /c rmdir "<symlink-path>"
```

### Linux / macOS — Symlink (Primary)

```bash
# Create symbolic link
ln -s "<source>" "<destination>"

# Example:
ln -s "/home/user/AI-Skills/react/component-patterns" ".github/skills/component-patterns"
```

**Verification:**

```bash
# Check if symlink
readlink "<path>"
# Or: test -L "<path>"
```

**Removal (safe — does NOT delete source):**

```bash
rm "<symlink-path>"
# Or for safety: unlink "<symlink-path>"
```

### All OS — Physical Copy (Fallback)

```powershell
# Windows
Copy-Item -Path "<source>" -Destination "<destination>" -Recurse
```

```bash
# Linux/macOS
cp -r "<source>" "<destination>"
```

**Removal:**

```powershell
# Windows
Remove-Item -Path "<destination>" -Recurse -Force
```

```bash
# Linux/macOS
rm -rf "<destination>"
```

---

## Linking Strategy for File Artifacts

When the artifact is a **single file** (e.g., `*.agent.md`, `*.prompt.md`):

- **If the target needs a different format** (see [agent-formats.md](agent-formats.md)):
  write a **converted copy**. Never link it — the target would get a format it does not
  read, and a link cannot be converted.
- **If the format already matches the target**: link the file (keeps it in sync) or copy
  it, as the user prefers.
- **If the file is in a folder with other assets**: link the entire parent folder (only
  when no conversion is needed).

### Single File Linking

**Windows — junctions do not work for files.** Try in this order:

```powershell
# 1. File symlink (needs the privilege described below)
cmd /c mklink "<destination-file>" "<source-file>"

# 2. Hardlink (no privilege needed; same volume only)
cmd /c mklink /H "<destination-file>" "<source-file>"

# 3. Copy (no sync)
Copy-Item "<source-file>" "<destination-file>"
```

**Linux/macOS:**

```bash
# Symlink works for files too
ln -s "<source-file>" "<destination-file>"
```

### Windows Symlink Privilege — Warn Before Falling Back

A file symlink fails with *"You do not have sufficient privilege to perform this
operation"* (`Carece de privilegios suficientes…`) unless one of these is true:

| Option                    | How                                                                                                   |
| ------------------------- | ----------------------------------------------------------------------------------------------------- |
| Developer Mode (simplest) | Settings → System → For developers → **Developer Mode** on                                            |
| Elevated prompt           | Run VS Code / the terminal as administrator (not recommended for daily work)                          |
| Local policy (Pro/Ent.)   | `secpol.msc` → Local Policies → User Rights Assignment → **Create symbolic links** → add user, sign out again |

Check Developer Mode before trying:

```powershell
(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock' -ErrorAction SilentlyContinue).AllowDevelopmentWithoutDevLicense -eq 1
```

If the symlink fails, **do not fall back silently**. Tell the user why it failed, show
the options above, and ask how to proceed:

```
header: Symlink not allowed
question: Windows did not allow a file symlink (Developer Mode is off).
          How do you want to install <file>?
options:
  - Hardlink     (stays in sync; see caveats)
  - Copy         (no sync; re-run the install to update)
  - Retry        (after enabling Developer Mode)
```

Hardlink caveats, which must be explained before using one:

- Source and destination must be on the **same volume** (e.g. both on `C:`). Check the
  drive letter first; if they differ, offer only copy or retry.
- The sync breaks silently when an editor or tool saves by **replacing** the file
  (write temp file + rename) instead of writing in place. The two files then diverge.
  Check with `fsutil hardlink list "<destination-file>"`: if it lists only one path, the
  link is broken.
- Deleting the hardlink does not delete the source, but editing it **does** change the
  central file.

### Recommendation

For **skills** (always folders): Use junction/symlink on the folder.
For **agents/prompts/instructions** (single files): converted copy when the format
differs; otherwise copy, or symlink if the user wants it kept in sync.

---

## Pre-Installation Checks

Before creating any link or copy:

1. **Source exists?** — Verify the source path is valid and accessible
2. **Destination already exists?**
   - Is it a junction/symlink pointing to the same source? → Skip (already installed)
   - Is it a junction/symlink pointing elsewhere? → Ask user: re-link or skip
   - Is it a regular folder/file? → Ask user: backup + replace, or skip
3. **Parent directory exists?** — Create with `mkdir -p` (Unix) or `New-Item -ItemType Directory -Force` (Windows)
4. **Path is inside project?** — NEVER create links outside the project working directory
5. **Circular reference?** — Ensure source is not inside destination or vice versa

---

## Post-Installation Verification

After creating each link:

| OS        | Verification Command             | Expected Result                       |
| --------- | -------------------------------- | ------------------------------------- |
| Windows   | `Test-Path "<dest>\SKILL.md"`    | `True`                                |
| Windows   | `(Get-Item "<dest>").Attributes` | Contains `ReparsePoint` for junctions |
| Linux/Mac | `test -e "<dest>/SKILL.md"`      | Exit code 0                           |
| Linux/Mac | `readlink "<dest>"`              | Shows source path for symlinks        |

---

## Safety Rules for Removal

### NEVER Delete Central Folder Content

Before executing any removal:

```powershell
# Windows — Validate path is inside project
$projectRoot = (Get-Location).Path
$targetPath = (Resolve-Path "<path-to-remove>").Path
if (-not $targetPath.StartsWith($projectRoot)) {
    Write-Error "REFUSED: Path is outside project directory"
    return
}
```

```bash
# Linux/macOS — Validate path is inside project
project_root=$(pwd)
target_path=$(realpath "<path-to-remove>")
if [[ "$target_path" != "$project_root"* ]]; then
    echo "REFUSED: Path is outside project directory"
    exit 1
fi
```

### Removal Decision Tree

1. Is it a junction/symlink? → Use `rmdir` (Win) or `rm`/`unlink` (Unix). Source is safe.
2. Is it a copy? → Use `rmdir /s /q` (Win) or `rm -rf` (Unix). Only deletes local copy.
   A converted agent is recognized by its `# Generated by ai-skills-management from …`
   comment; delete the single file with `del` (Win) or `rm` (Unix).
   A hardlink (`fsutil hardlink list` shows 2+ paths) is removed with `del` / `rm`; the
   central file is kept.
3. Is it neither? → Do NOT remove. Alert user that this is not a managed artifact.
4. Does the path resolve outside the project? → REFUSE. Alert user.

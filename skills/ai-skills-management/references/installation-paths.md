# Installation Paths Reference

Complete path matrix for linking skills, agents, prompts, and instructions per AI agent and OS.

---

## Destination Paths by Agent

### GitHub Copilot (VS Code)

| Artifact Type | Project Path                                  | Notes                                   |
| ------------- | --------------------------------------------- | --------------------------------------- |
| Skill         | `.github/skills/<skill-name>/`                | Folder containing `SKILL.md`            |
| Agent         | `.github/agents/<name>.agent.md`              | Single file, copy or link parent folder |
| Prompt        | `.github/prompts/<name>.prompt.md`            | Single file                             |
| Instruction   | `.github/instructions/<name>.instructions.md` | Single file                             |
| Context       | `.github/copilot-instructions.md`             | Do NOT overwrite; append or skip        |

### Claude Code

| Artifact Type  | Project Path                   | Notes                            |
| -------------- | ------------------------------ | -------------------------------- |
| Skill          | `.claude/skills/<skill-name>/` | Folder containing `SKILL.md`     |
| Command/Prompt | `.claude/commands/<name>.md`   | Single markdown file             |
| Context        | `CLAUDE.md` (root)             | Do NOT overwrite; append or skip |
| Settings       | `.claude/settings.json`        | Do NOT overwrite                 |

### Codex (OpenAI)

| Artifact Type | Project Path                  | Notes                        |
| ------------- | ----------------------------- | ---------------------------- |
| Skill         | `.codex/skills/<skill-name>/` | Folder containing `SKILL.md` |
| Instructions  | `.codex/instructions.md`      | Single file; append only     |
| Context       | `AGENTS.md` (root)            | Do NOT overwrite             |

### Gemini CLI

| Artifact Type | Project Path                   | Notes                        |
| ------------- | ------------------------------ | ---------------------------- |
| Skill         | `.gemini/skills/<skill-name>/` | Folder containing `SKILL.md` |
| Context       | `GEMINI.md` (root)             | Do NOT overwrite             |
| Settings      | `.gemini/settings.json`        | Do NOT overwrite             |

### OpenCode

| Artifact Type | Project Path                     | Notes                        |
| ------------- | -------------------------------- | ---------------------------- |
| Skill         | `.opencode/skills/<skill-name>/` | Folder containing `SKILL.md` |
| Agent         | `.opencode/agents/<name>.md`     | Single file                  |
| Context       | `opencode.json`                  | Do NOT overwrite             |

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

- **If the file is alone in its source folder**: Link or copy the individual file
- **If the file is in a folder with other assets**: Link the entire parent folder

### Single File Linking

**Windows (junction not available for files — use copy or hardlink):**

```powershell
# Hardlink (file only, same volume required)
cmd /c mklink /H "<destination-file>" "<source-file>"

# Or just copy
Copy-Item "<source-file>" "<destination-file>"
```

**Linux/macOS:**

```bash
# Symlink works for files too
ln -s "<source-file>" "<destination-file>"
```

### Recommendation

For **skills** (always folders): Use junction/symlink on the folder.
For **agents/prompts/instructions** (often single files): Prefer copying the file, or symlink if the user prefers to keep them in sync.

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
3. Is it neither? → Do NOT remove. Alert user that this is not a managed artifact.
4. Does the path resolve outside the project? → REFUSE. Alert user.

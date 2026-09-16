---
name: ai-skills-management
description: >
  Manage, discover, and install shared AI skills, agents, prompts, and instructions
  from a central folder into any project. Supports GitHub Copilot, Claude Code, Codex,
  Gemini CLI, OpenCode, and other AI agents. Use when the user asks to: setup AI for
  this project, configure AI tooling for this repository, install shared AI skills into
  this project, initialize AI environment for this repo, connect this project to shared
  AI skills, configure copilot and claude skills for this solution, setup reusable AI
  prompts and skills, link shared development skills to this repository, prepare this
  project for AI-assisted development, install enterprise AI coding skills, discover
  available AI tools, manage AI skills, list installed AI skills, remove AI skills from
  project, scan AI environment, detect AI agents.
---

# AI Skills Management

Manage shared AI skills, agents, prompts, and instructions from a central folder.
Link them into any project for any AI agent without copying files manually.

## When to Use

Use this skill when the user wants to:

- Setup or configure AI tooling for a project
- Install or link shared AI skills into a repository
- Initialize an AI development environment
- Discover available skills, agents, or prompts
- Connect a project to a shared skills folder
- Remove or unlink skills from a project
- Scan what AI tools and agents are available

---

## Workflow

Execute these steps sequentially. Show progress to the user after each step.

### Step 1 — Ask for Central Skills Folder

> **STOP. Do NOT proceed to Step 2 until the user provides this path.**

Use the agent's **interactive question tool** (e.g. `vscode_askQuestions`, `ask_followup_question`, or equivalent) to present a focused input prompt — not a chat message. The user should be able to type the path directly into the question and continue without sending a separate reply.

**Question to present:**

```
header: Central Skills Folder
question: What is the path to your central AI skills folder?
         (where all your shared skills, agents and prompts are stored)
placeholder / example: C:\AI-Skills   or   ~/ai-skills
```

**NEVER:**

- Assume a default path
- Infer the path from the environment, history, or any known folder
- Use paths like `~/.agents/skills/`, `~/.copilot/skills/`, `C:\Users\...\`, or any other location without explicit confirmation
- Skip this step or proceed without a confirmed path

**Validation before moving to Step 2:**

- The path exists and is a directory → continue
- The path does not exist → present a second question: "That folder doesn't exist. Create it now?" (Yes / No). If No, ask again for a valid path
- Echo the confirmed path once: `✓ Central folder: <path>` — then immediately continue

---

### Step 2 — Detect Current Project

Scan the working directory to identify the project stack. Show each detection as it happens.

**Look for these indicators:**

| File / Pattern                                     | Stack                                                                   |
| -------------------------------------------------- | ----------------------------------------------------------------------- |
| `package.json`                                     | Node.js (inspect `dependencies` for React, Vue, Angular, Next.js, etc.) |
| `*.sln`                                            | .NET Solution                                                           |
| `*.csproj`                                         | .NET Project (inspect for WinForms, WPF, Blazor, Web API, etc.)         |
| `pom.xml`                                          | Java (Maven)                                                            |
| `build.gradle` / `build.gradle.kts`                | Java/Kotlin (Gradle)                                                    |
| `requirements.txt` / `pyproject.toml` / `setup.py` | Python                                                                  |
| `go.mod`                                           | Go                                                                      |
| `Cargo.toml`                                       | Rust                                                                    |
| `composer.json`                                    | PHP                                                                     |
| `*.swift` / `Package.swift`                        | Swift                                                                   |
| `pubspec.yaml`                                     | Flutter/Dart                                                            |

**Output format** (show progressively):

```
Detecting project...
  ✓ Found package.json → Node.js
  ✓ Dependencies: React 19, TypeScript, Vite
  ✓ Stack identified: React + TypeScript
```

For detailed detection rules and sub-detection logic, see [references/project-detection.md](references/project-detection.md).

---

### Step 3 — Detect AI Tools and Agents

Scan both the project and the system for AI-related tools, configurations, and agents.
Show findings progressively as they are discovered.

**At project level**, look for:

- `.github/` → GitHub Copilot (check for `copilot-instructions.md`, `agents/`, `skills/`, `prompts/`, `instructions/`)
- `.claude/` → Claude Code (check for `settings.json`, `commands/`, `skills/`)
- `.codex/` → Codex (check for `instructions.md`)
- `.gemini/` → Gemini CLI
- `.opencode/` → OpenCode
- `.continue/` → Continue
- `.clinerules` / `.roo/` → Cline / Roo
- `.aider*` → Aider
- `AGENTS.md` / `CLAUDE.md` / `CONVENTIONS.md`
- `.mcp.json` / `mcp.json` → MCP servers
- `.vscode/settings.json` → VS Code AI settings

**At system level**, verify:

- CLI tools: `claude`, `codex`, `gemini`, `opencode`, `aider`, `continue`
- VS Code extensions: run `code --list-extensions` and filter AI-related
- Global config folders (OS-dependent)

**Output format:**

```
Scanning AI environment...
  ✓ Detected: VS Code
  ✓ Detected: GitHub Copilot (extension)
  ✓ Detected: Claude Code (CLI)
  ✓ Detected: .github/ folder (existing config)
  ✓ Detected: .claude/ folder (existing config)
  ✓ Detected: AGENTS.md (project context)
  ✗ Not found: Codex CLI
  ✗ Not found: Gemini CLI
```

For the complete detection matrix, see [references/agent-detection.md](references/agent-detection.md).

#### No Agents Detected at Project Level

If **none** of the agent configuration folders or files are found at project level (no `.github/`, `.claude/`, `.codex/`, `.gemini/`, `.opencode/`, etc.), use the agent's **interactive question tool** (`vscode_askQuestions`, `ask_followup_question`, or equivalent) to present a **multi-select** prompt:

```
header: AI Agents to Configure
question: No AI agent configuration was found in this project.
          Which agent(s) do you want to set up?
multiSelect: true
options:
  - GitHub Copilot   (creates .github/)
  - Claude Code      (creates .claude/)
  - Codex            (creates .codex/)
  - Gemini CLI       (creates .gemini/)
  - OpenCode         (creates .opencode/)
```

**Rules:**

- Pre-select agents whose system-level tools were detected (e.g., if `claude` CLI is installed, pre-select Claude Code). If nothing was found at system level, present all options with no pre-selection
- Store the confirmed selection as the list of **target agents** for Step 7

**Echo after selection:**

```
✓ Agents to configure: GitHub Copilot, Claude Code
  → Will create: .github/  .claude/
```

---

### Step 4 — Scan Available Skills in Central Folder

Recursively scan the central folder. Classify each artifact found by type.

**Detection rules:**

| File found                | Classification |
| ------------------------- | -------------- |
| `SKILL.md`                | Skill          |
| `*.agent.md`              | Agent          |
| `*.prompt.md`             | Prompt         |
| `*.instructions.md`       | Instruction    |
| `AGENTS.md` / `CLAUDE.md` | Context file   |

**Build a categorized tree and show it:**

```
Central folder: C:\AI-Skills\

  react/ (4 skills, 1 agent)
    ├── component-patterns/  [skill]
    ├── api-handling/        [skill]
    ├── testing/             [skill]
    ├── forms/               [skill]
    └── code-reviewer.agent.md [agent]

  dotnet/ (3 skills)
    ├── clean-architecture/  [skill]
    ├── winforms-mvp/        [skill]
    └── entity-framework/    [skill]

  shared/ (2 skills, 1 prompt)
    ├── git-workflow/         [skill]
    ├── code-review/         [skill]
    └── commit-message.prompt.md [prompt]

  Total: 9 skills, 1 agent, 1 prompt
```

---

### Step 5 — Recommend Skills and Agents by Stack

Cross-reference the detected stack (Step 2) with available categories (Step 4).
Recommend **both skills and agents** (and any prompts/instructions) that match the stack.

> **Recommending is NOT installing.** This step only *displays* suggestions. It must
> never install anything, and the ★ marks are purely informational. Installation happens
> only in Step 7, and only for items the user explicitly selects in Step 6. Do not skip
> Step 6 or treat these recommendations as an implicit selection.

> **CRITICAL — derive recommendations from the DETECTED stack, never from the examples below.**
> Look up the stack you actually detected in Step 2 in the matrix, then mark ★ only the
> categories that match it. The code blocks in this step are **format samples only** —
> do NOT recommend React categories for a .NET project (or vice versa). If you detected
> .NET, recommend `dotnet/`, `api/`, `sql/`, `shared/`, `testing/`, etc. — not `react/`.

**Recommendation matrix:**

| Detected Stack                  | Recommended Categories                       |
| ------------------------------- | -------------------------------------------- |
| React / Next.js / Vue / Angular | `react/`, `frontend/`, `shared/`, `testing/` |
| .NET (Web API, Blazor)          | `dotnet/`, `api/`, `shared/`, `sql/`, `testing/` |
| .NET (WinForms, WPF)            | `winforms/`, `dotnet/`, `sql/`, `shared/`    |
| Java                            | `java/`, `shared/`, `sql/`, `testing/`       |
| Python                          | `python/`, `shared/`, `testing/`             |
| Go / Rust                       | `go/`, `rust/`, `shared/`                    |

For the full, authoritative mapping (including sub-stacks like Web API vs WinForms vs
Blazor), always consult [references/project-detection.md](references/project-detection.md).

Present recommendations grouped by artifact type. Use the **generic format** below and
fill it with the categories that match *your* detected stack — the placeholders are not
tied to any specific technology, so build the list from the matrix, never from a fixed
example.

```
Recommended for your <DETECTED_STACK> project:

  Skills:
    ★ <matching-category>/<skill-name>   [skill]
    ★ shared/<skill-name>                [skill]
    ...

  Agents:      (only if the central folder has agents in matching categories)
    ★ <matching-category>/<name>.agent.md [agent]

  Prompts:     (only if present in matching categories)
    ★ shared/<name>.prompt.md            [prompt]

Also available (other categories):
  · <non-matching-category>/<name>       [skill]
```

Agents are recommended using the same category matching as skills: an agent inside a
recommended category is marked with ★. Agents in non-matching categories are still
listed under "Also available".

For full recommendation rules, see [references/project-detection.md](references/project-detection.md).

---

### Step 6 — Interactive Selection

Let the user choose what to install. Selections may include **skills, agents, prompts,
and instructions** — all artifact types are installable, not only skills.

**Present each artifact type in its own separate dialog** — do NOT mix skills and agents
in a single list distinguished only by a `[skill]` / `[agent]` tag. Use one dedicated
multi-select prompt per artifact type so the user reasons about them independently.

> **CRITICAL — nothing is installed unless the user explicitly selects it.**
> Do NOT pre-check/pre-select any option. Recommended items (★) are only *labeled* as
> recommended; they must start **unchecked**. If the user confirms a dialog with an
> empty selection, install **nothing** from it — never fall back to the recommended
> defaults. Recommendations are hints, not an implicit selection.

#### 6.1 — Select Skills

Use the agent's **interactive question tool** (`vscode_askQuestions`,
`ask_followup_question`, or equivalent) with a multi-select prompt that lists **only
skills**. Mark the recommended ones with a `(recommended)` label, but leave every option
**unchecked** by default:

```
header: Skills to Install
question: Which skills do you want to install?
          (nothing is pre-selected — pick the ones you want; ★ = recommended for your stack)
multiSelect: true
options:
  - <matching-category>/<skill-name>   ★ recommended
  - shared/<skill-name>                ★ recommended
  - <other-category>/<skill-name>
```

If the user selects no skills, skip skill installation entirely.

#### 6.2 — Select Agents

Only if the central folder contains agents, present a **second, separate** multi-select
prompt that lists **only agents**. Mark recommended ones with a label, but leave every
option **unchecked** by default:

```
header: Agents to Install
question: Which agents do you want to install?
          (nothing is pre-selected — pick the ones you want; ★ = recommended for your stack)
multiSelect: true
options:
  - <matching-category>/<name>.agent.md   ★ recommended
  - <other-category>/<name>.agent.md
```

> If there are no agents in the central folder, skip this dialog entirely. The same
> applies to prompts/instructions: present them in their own dialog only when present.
> If the user selects no agents, skip agent installation entirely.

#### 6.3 — Quick Options

Offer these shortcuts, but apply one **only if the user explicitly chooses it**. Never
apply a shortcut on your own — the default when the user picks nothing is to install
nothing:

- **All recommended** — Everything marked ★ across all types
- **All available** — Everything in the central folder
- **By category** — All items from selected categories

#### 6.4 — Confirm Target Agents

Also confirm which **target agents** to install for:

- If **no agents were detected** in Step 3: the user already chose agents there — skip this sub-step and use that selection directly
- If agents **were detected** in Step 3: use the agent's **interactive question tool** (`vscode_askQuestions`, `ask_followup_question`, or equivalent) to let the user confirm or adjust, with all detected agents pre-selected:

```
header: Target Agents
question: Install skills for which agents?
          (all detected agents are pre-selected — deselect any you want to skip)
multiSelect: true
options:
  - GitHub Copilot   (pre-selected if .github/ was found)
  - Claude Code      (pre-selected if .claude/ was found)
  - Codex            (pre-selected if .codex/ was found)
  - Gemini CLI       (pre-selected if .gemini/ was found)
  - OpenCode         (pre-selected if .opencode/ was found)
```

**Echo after confirmation:**

```
✓ Installing for: GitHub Copilot, Claude Code
```

---

### Step 7 — Install Selected Skills and Agents

> **STOP if nothing was selected.** Build the install set only from what the user
> explicitly checked in Step 6 (skills, agents, prompts, instructions). If that combined
> set is empty, install **nothing**, tell the user "No items selected — nothing to
> install", and end here. Never install the stack-recommended defaults on an empty
> selection.

For each selected item, link it into the project for each target agent. Handle each
artifact type according to its shape:

- **Skills** are folders (contain `SKILL.md`) → junction/symlink the folder
- **Agents / prompts / instructions** are single files (`*.agent.md`, `*.prompt.md`,
  `*.instructions.md`) → copy the file, or symlink it if the user wants it kept in sync

#### 7.1 — Detect OS and Choose Link Method

| OS      | Primary Method | Command                                | Fallback                         |
| ------- | -------------- | -------------------------------------- | -------------------------------- |
| Windows | Junction       | `cmd /c mklink /J "<dest>" "<source>"` | Symlink (`mklink /D`), then copy |
| Linux   | Symlink        | `ln -s "<source>" "<dest>"`            | Copy (`cp -r`)                   |
| macOS   | Symlink        | `ln -s "<source>" "<dest>"`            | Copy (`cp -r`)                   |

#### 7.2 — Determine Destination Paths

For each selected item, create links in the appropriate location per agent.
See [references/installation-paths.md](references/installation-paths.md) for the complete path matrix.

**Quick reference:**

| Agent    | Skills                     | Agents              | Prompts             | Instructions            |
| -------- | -------------------------- | ------------------- | ------------------- | ----------------------- |
| Copilot  | `.github/skills/<name>/`   | `.github/agents/`   | `.github/prompts/`  | `.github/instructions/` |
| Claude   | `.claude/skills/<name>/`   | `.claude/commands/` | `.claude/commands/` | `.claude/`              |
| Codex    | `.codex/skills/<name>/`    | `.codex/`           | `.codex/`           | `.codex/`               |
| Gemini   | `.gemini/skills/<name>/`   | `.gemini/`          | `.gemini/`          | `.gemini/`              |
| OpenCode | `.opencode/skills/<name>/` | `.opencode/agents/` | `.opencode/`        | `.opencode/`            |

#### 7.3 — Execute Installation

For each item × each target agent:

1. **Resolve the destination** based on artifact type and target agent, using the
   path matrix in [references/installation-paths.md](references/installation-paths.md).
   Note that agents map to a different folder per agent (e.g. `.github/agents/`,
   `.claude/commands/`, `.opencode/agents/`).
2. **Check if destination already exists**
   - If junction/symlink → skip (already linked) or ask to re-link
   - If regular folder/file → ask user: overwrite, skip, or backup
3. **Create parent directories** if they don't exist
4. **Create link/copy**
   - Folders (skills): junction (Windows) or symlink (Unix)
   - Single files (agents/prompts/instructions): copy, or symlink to keep in sync
5. **Verify** the link/copy works (check target is readable)

**Output format:**

```
Installing skills and agents...

  Skills:
    react/component-patterns
      ✓ .github/skills/component-patterns → C:\AI-Skills\react\component-patterns [junction]
      ✓ .claude/skills/component-patterns → C:\AI-Skills\react\component-patterns [junction]

    shared/code-review
      ✓ .github/skills/code-review → C:\AI-Skills\shared\code-review [junction]
      ✓ .claude/skills/code-review → C:\AI-Skills\shared\code-review [junction]

  Agents:
    react/code-reviewer.agent.md
      ✓ .github/agents/code-reviewer.agent.md → C:\AI-Skills\react\code-reviewer.agent.md [copy]
      ✓ .claude/commands/code-reviewer.md → C:\AI-Skills\react\code-reviewer.agent.md [copy]

Installed: 2 skills + 1 agent for 2 agents (6 artifacts created)
```

> **Note on agent file naming:** Copilot and OpenCode keep the `.agent.md` /
> `.md` extension in `agents/`; Claude Code places agents as slash commands under
> `.claude/commands/<name>.md`. Adjust the destination filename per agent as shown
> in the path matrix.

---

### Step 8 — Summary

Present a final summary:

```
AI Skills Management — Complete

  Project: MyApp (React + TypeScript)
  Central folder: C:\AI-Skills\
  Link method: Junction (Windows)

  Installed:
    3 skills → .github/skills/ (GitHub Copilot)
    1 agent  → .github/agents/ (GitHub Copilot)
    3 skills → .claude/skills/ (Claude Code)
    1 agent  → .claude/commands/ (Claude Code)

  Detected agents: Copilot, Claude Code
  Available in central: 9 skills, 1 agent, 1 prompt

  To remove links later, use: "remove AI skills from this project"
```

---

## Unlink / Remove Skills and Agents

When the user asks to remove or unlink skills or agents:

1. **Scan project** for existing links:
   - Skills: `.github/skills/`, `.claude/skills/`, `.codex/skills/`, `.gemini/skills/`, `.opencode/skills/`
   - Agents: `.github/agents/`, `.claude/commands/`, `.opencode/agents/`
2. **Identify type**: junction, symlink, or copy (use `fsutil reparsepoint query` on Windows, `readlink` on Unix)
3. **Show what's installed** (skills and agents) and let the user select what to remove
4. **Remove safely:**
   - Junction (Windows): `cmd /c rmdir "<path>"` — does NOT delete original
   - Symlink (Unix): `rm "<path>"` — does NOT delete original
   - Copy (folder): `rm -rf "<path>"` (Unix) / `rmdir /s /q "<path>"` (Windows) — deletes local copy only
   - Copy (single file, e.g. an agent): `rm "<path>"` (Unix) / `del "<path>"` (Windows) — deletes local copy only

### CRITICAL SAFETY RULES

- **NEVER** delete, modify, or write to the central skills folder
- **NEVER** run `rm -rf` or `rmdir /s` on a path outside the current project
- **ALWAYS** verify the target path is INSIDE the project working directory before deletion
- **ALWAYS** confirm with the user before removing anything
- If a path resolves to a location outside the project, REFUSE and alert the user

---

## List Installed Skills and Agents

When the user asks to list or show installed AI skills or agents:

1. Scan all agent skill directories (`.github/skills/`, `.claude/skills/`, etc.) and
   agent directories (`.github/agents/`, `.claude/commands/`, `.opencode/agents/`)
2. For each entry, detect if it's a junction, symlink, or copy
3. Show the source (if linked) or mark as local copy
4. Group by agent, and within each agent by artifact type (skills, agents)

---

## Notes

- This skill uses ONLY folder-based organization. No package managers, registries, or services.
- All operations are local and offline.
- The central folder structure is flexible — scanning is recursive and depth-independent.
- Adding support for a new agent only requires updating the path tables.

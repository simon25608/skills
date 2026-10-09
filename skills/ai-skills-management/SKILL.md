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

### Interactive Question Tool — Limits

Every step that asks the user something uses the agent's interactive question tool.
These tools differ, so adapt to the one available:

- **No free-text field** (e.g. Claude Code's `AskUserQuestion` only offers fixed options
  plus an "Other" box): do NOT ask for a path or other free text through it with
  placeholder options like "Type the path". Ask for it **in a chat message** and wait
  for the reply. Use the tool only for choices.
- **Option limits** (e.g. `AskUserQuestion`: 2–4 options per question, up to 4
  questions per call): when a list is longer, first show the **full list in chat**
  (grouped and numbered), then split the options into several questions of the same
  type (`Skills 1/3`, `Skills 2/3`…) ordered by priority. Tell the user they can add
  items not shown by number or name through "Other".
- **No pre-selection support**: state in the question text which options are
  recommended or detected, instead of pre-selecting them.
- **No interactive tool at all**: ask in chat with a numbered list and accept answers
  like `1, 3, 5` or `all recommended`.

### Step 1 — Ask for Central Skills Folder

> **STOP. Do NOT proceed to Step 2 until the user provides this path.**

Use the agent's **interactive question tool** (e.g. `vscode_askQuestions`, `ask_followup_question`, or equivalent) to present a focused input prompt — not a chat message. The user should be able to type the path directly into the question and continue without sending a separate reply.

If the tool has no free-text input (see *Interactive Question Tool — Limits*), ask for the
path in a chat message instead and stop until the user replies. If an answer comes back
without a path (e.g. an option was picked but nothing was typed), ask again — never
continue without a real path.

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

- `.github/` → GitHub Copilot **only if** it has `copilot-instructions.md`, `agents/`, `skills/`, `prompts/` or `instructions/` (a `.github/` with only `workflows/` is not Copilot)
- `.claude/` → Claude Code (check for `settings.json`, `agents/`, `commands/`, `skills/`)
- `.codex/` → Codex (check for `config.toml`, `agents/`); `.agents/skills/` → Codex/Gemini/OpenCode skills
- `.gemini/` → Gemini CLI (check for `settings.json`, `agents/`, `commands/`, `skills/`)
- `.opencode/` → OpenCode (check for `agents/`, `commands/`, `skills/`)
- `.continue/` → Continue
- `.clinerules` / `.roo/` → Cline / Roo
- `.aider*` → Aider
- `AGENTS.md` / `CLAUDE.md` / `CONVENTIONS.md`
- `.mcp.json` / `mcp.json` → MCP servers
- `.vscode/settings.json` → VS Code AI settings

**At system level**, verify:

- CLI tools: `claude`, `codex`, `gemini`, `opencode`, `aider`, `continue`
- VS Code extensions: run `code --list-extensions` and filter AI-related, comparing IDs
  case-insensitively (`anthropic.claude-code`, `openai.chatgpt`, `sst-dev.opencode`,
  `github.copilot-chat`, …). An extension counts as the tool being installed even if its
  CLI is not on the `PATH`
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

| File found                                                        | Classification |
| ----------------------------------------------------------------- | -------------- |
| `SKILL.md`                                                        | Skill          |
| `*.agent.md`                                                      | Agent (Copilot format) |
| `*.md` inside an `agents/` folder with `name` + `description` frontmatter | Agent (Claude, Gemini or OpenCode format) |
| `*.toml` inside an `agents/` folder with `developer_instructions` | Agent (Codex format) |
| `*.prompt.md`                                                     | Prompt         |
| `*.instructions.md`                                               | Instruction    |
| `AGENTS.md` / `CLAUDE.md`                                         | Context file   |

Files inside a skill folder (e.g. a skill's own `AGENTS.md` or `agents/`) belong to that
skill — do not list them as separate artifacts.

For every skill and agent, also read the frontmatter `name` and `description` (not the
body). They are needed for the recommendations in Step 5, and for agents, to detect
their source format (see [references/agent-formats.md](references/agent-formats.md)).
Show the format next to each agent: `code-reviewer.agent.md [agent · Copilot format]`.

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

Cross-reference the detected stack (Step 2) with available categories (Step 4) **and
with each artifact's `description`**. Recommend **both skills and agents** (and any
prompts/instructions) that match the stack.

Category folders (`react/`, `dotnet/`…) are optional: when the central folder is flat
(`skills/`, `agents/`) or a category says nothing about the stack, the description is
the only signal. Always apply the description-based matching in
[references/project-detection.md](references/project-detection.md#description-based-matching):
match the `name` and `description` against the project's framework, language,
dependencies and internal packages, and show the matched keyword next to each
recommendation (`★ bcp-data-patterns — matches @ba-react/bcp-data`).

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

Agents are recommended using the same category and description matching as skills: an
agent inside a recommended category, or whose description matches the stack, is marked
with ★. Other agents are still listed under "Also available".

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

- **Skills** are folders (contain `SKILL.md`) → junction/symlink the folder. The
  `SKILL.md` format is the same for every tool, so skills are never converted
- **Agents** are single files whose format differs per tool → install as the target's
  **agent** (subagent) type, converting the frontmatter when the source format differs
  from the target (see 7.3)
- **Prompts / instructions** are single files → copy the file, or symlink it if the user
  wants it kept in sync (convert for Gemini, which uses TOML commands)

#### 7.1 — Detect OS and Choose Link Method

| OS      | Primary Method | Command                                | Fallback                         |
| ------- | -------------- | -------------------------------------- | -------------------------------- |
| Windows | Junction       | `cmd /c mklink /J "<dest>" "<source>"` | Symlink (`mklink /D`), then copy |
| Linux   | Symlink        | `ln -s "<source>" "<dest>"`            | Copy (`cp -r`)                   |
| macOS   | Symlink        | `ln -s "<source>" "<dest>"`            | Copy (`cp -r`)                   |

Junctions only work for folders. A **file** symlink on Windows needs Developer Mode, an
elevated prompt or the "Create symbolic links" right. If it fails, do **not** fall back
silently to a hardlink or copy: explain why it failed, how to enable it, and the
hardlink caveats, then ask the user (see
[references/installation-paths.md](references/installation-paths.md#windows-symlink-privilege--warn-before-falling-back)).

#### 7.2 — Determine Destination Paths

For each selected item, create links in the appropriate location per agent.
See [references/installation-paths.md](references/installation-paths.md) for the complete path matrix.

**Quick reference:**

| Agent    | Skills                       | Agents                        | Prompts                         | Instructions            |
| -------- | ---------------------------- | ----------------------------- | ------------------------------- | ----------------------- |
| Copilot  | `.github/skills/<name>/`     | `.github/agents/<name>.agent.md` | `.github/prompts/`           | `.github/instructions/` |
| Claude   | `.claude/skills/<name>/`     | `.claude/agents/<name>.md`    | `.claude/commands/<name>.md`    | `CLAUDE.md` (append)    |
| Codex    | `.agents/skills/<name>/`     | `.codex/agents/<name>.toml`   | — (offer as skill)              | `AGENTS.md` (append)    |
| Gemini   | `.gemini/skills/<name>/`     | `.gemini/agents/<name>.md`    | `.gemini/commands/<name>.toml`  | `GEMINI.md` (append)    |
| OpenCode | `.opencode/skills/<name>/`   | `.opencode/agents/<name>.md`  | `.opencode/commands/<name>.md`  | `AGENTS.md` (append)    |

> **Agents go to the target's agents folder — never to `commands/`.** In Claude Code a
> command (`.claude/commands/`) is a prompt the user runs with `/name`; an agent
> (`.claude/agents/`) is a subagent Claude delegates to.

Several tools share skill folders (OpenCode reads `.claude/skills/` and `.agents/skills/`;
Codex and Gemini read `.agents/skills/`). Do not link the same skill twice for a target
that already sees it — see
[references/installation-paths.md](references/installation-paths.md#shared-skill-folders-avoid-duplicates).

#### 7.3 — Execute Installation

For each item × each target agent:

1. **Resolve the destination** based on artifact type and target agent, using the
   path matrix in [references/installation-paths.md](references/installation-paths.md).
   Agents map to a different folder and format per agent (`.github/agents/*.agent.md`,
   `.claude/agents/*.md`, `.codex/agents/*.toml`, `.gemini/agents/*.md`,
   `.opencode/agents/*.md`).
2. **For agents, compare formats** using
   [references/agent-formats.md](references/agent-formats.md):
   - Source format = target format → link or copy the file as-is.
   - Source format ≠ target format → build a **converted copy**: translate `name`,
     `tools` and `model`, drop keys the target does not support, keep the body, and add
     the `# Generated by ai-skills-management from <source>` provenance comment.
   - Show the conversion report (mapped and dropped tools, model change) and ask for
     confirmation when something was dropped or mapped with a fallback.
   - Converted copies do not stay in sync. If the user chose "keep in sync" for an agent
     that needs conversion, explain this and offer a converted copy instead.
3. **Check if destination already exists**
   - If junction/symlink → skip (already linked) or ask to re-link
   - If it is a converted copy with the provenance comment for the same source → ask to
     regenerate or skip
   - If regular folder/file → ask user: overwrite, skip, or backup
4. **Create parent directories** if they don't exist
5. **Create link/copy**
   - Folders (skills): junction (Windows) or symlink (Unix)
   - Agents: converted copy, or link/copy when the format already matches
   - Prompts/instructions: copy, or symlink to keep in sync
6. **Verify** the link/copy works (the target is readable and, for agents, the
   frontmatter has the fields the target requires)

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
    react/code-reviewer.agent.md  (Copilot format)
      ✓ .github/agents/code-reviewer.agent.md → C:\AI-Skills\react\code-reviewer.agent.md [copy]
      ✓ .claude/agents/code-reviewer.md ← converted from Copilot format [converted copy]
          model: Claude Sonnet 4.6 (copilot) → sonnet
          tools: 17 → Read, Grep, Glob, LSP, Bash, Agent  (dropped: read/terminalSelection, read/terminalLastCommand)

Installed: 2 skills + 1 agent for 2 agents (6 artifacts created)
```

> **Note on agent file naming:** Copilot keeps `.agent.md` in `.github/agents/`;
> Claude, Gemini and OpenCode use `<name>.md` in their `agents/` folder; Codex uses
> `<name>.toml` in `.codex/agents/`. Use the slug from
> [references/agent-formats.md](references/agent-formats.md#5-name-mapping) as `<name>`.

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
    1 agent  → .claude/agents/ (Claude Code, converted copy)

  Detected agents: Copilot, Claude Code
  Available in central: 9 skills, 1 agent, 1 prompt

  Notes:
    · Converted agents do not stay in sync — re-run the install after changing the source
    · Codex only runs custom agents when you ask for them explicitly
    · Restart the agent tool if it was open, so it picks up the new agents folder

  To remove links later, use: "remove AI skills from this project"
```

Include in **Notes** only the lines that apply (e.g. the Codex line only if Codex was a
target), plus any warning raised during installation (hardlink used, tools dropped,
duplicate skill folders).

---

## Unlink / Remove Skills and Agents

When the user asks to remove or unlink skills or agents:

1. **Scan project** for existing links:
   - Skills: `.github/skills/`, `.claude/skills/`, `.agents/skills/`, `.gemini/skills/`, `.opencode/skills/`
   - Agents: `.github/agents/`, `.claude/agents/`, `.codex/agents/`, `.gemini/agents/`, `.opencode/agents/`
   - Prompts: `.github/prompts/`, `.claude/commands/`, `.gemini/commands/`, `.opencode/commands/`
   - Legacy: agents installed by older versions of this skill in `.claude/commands/`
     (frontmatter with Copilot `tools`/`model`) — offer to move them to `.claude/agents/`
2. **Identify type**: junction, symlink, hardlink, converted copy, or copy (use `fsutil reparsepoint query` / `fsutil hardlink list` on Windows, `readlink` on Unix; a converted copy has the `# Generated by ai-skills-management from …` comment)
3. **Show what's installed** (skills and agents) and let the user select what to remove
4. **Remove safely:**
   - Junction (Windows): `cmd /c rmdir "<path>"` — does NOT delete original
   - Symlink (Unix): `rm "<path>"` — does NOT delete original
   - Copy (folder): `rm -rf "<path>"` (Unix) / `rmdir /s /q "<path>"` (Windows) — deletes local copy only
   - Copy (single file, e.g. an agent): `rm "<path>"` (Unix) / `del "<path>"` (Windows) — deletes local copy only
   - Hardlink (Windows): `del "<path>"` — removes the project entry only; the central file is kept

### CRITICAL SAFETY RULES

- **NEVER** delete, modify, or write to the central skills folder
- **NEVER** run `rm -rf` or `rmdir /s` on a path outside the current project
- **ALWAYS** verify the target path is INSIDE the project working directory before deletion
- **ALWAYS** confirm with the user before removing anything
- If a path resolves to a location outside the project, REFUSE and alert the user

---

## List Installed Skills and Agents

When the user asks to list or show installed AI skills or agents:

1. Scan all agent skill directories (`.github/skills/`, `.claude/skills/`, `.agents/skills/`, etc.) and
   agent directories (`.github/agents/`, `.claude/agents/`, `.codex/agents/`, `.gemini/agents/`, `.opencode/agents/`)
2. For each entry, detect if it's a junction, symlink, hardlink, converted copy (and its source), or copy
3. Show the source (if linked) or mark as local copy
4. Group by agent, and within each agent by artifact type (skills, agents)

---

## Notes

- This skill uses ONLY folder-based organization. No package managers, registries, or services.
- All operations are local and offline.
- The central folder structure is flexible — scanning is recursive and depth-independent.
- Adding support for a new agent requires updating the path tables and, for agents, the
  format and tool mapping in [references/agent-formats.md](references/agent-formats.md).

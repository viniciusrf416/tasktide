# TaskTide

**Model routing for Claude Code.** A skill and seven subagents that send each development task to the cheapest model that can do it safely: Haiku to map and coordinate, Sonnet to implement and review, Opus only for architecture and high-risk work.

![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
![Claude Code](https://img.shields.io/badge/Claude%20Code-skill%20%2B%20agents-8A2BE2.svg)
![Models](https://img.shields.io/badge/models-Haiku%20%C2%B7%20Sonnet%20%C2%B7%20Opus-orange.svg)

---

## Contents

```
.claude/
  skills/tasktide/SKILL.md
  agents/
    haiku-coordinator.md     haiku-implementer.md
    sonnet-implementer.md    sonnet-reviewer.md
    opus-architect.md        opus-implementer.md
    opus-reviewer.md
```

The package is Markdown only. It has no scripts, hooks or dependencies.

## Quick start

```text
/tasktide Implement password recovery in the authentication module
```

You can also leave the command out: the skill loads automatically when a request matches its description.

## Installation

Extract the package **outside** `.claude/skills/`. If the skill sits in a nested folder, Claude Code will not find it.

### Per project

Copy the package's `.claude/` folder to the repository root. If the project already has a `.claude/` folder, copy `skills/tasktide/` and the files in `agents/` into it.

### Personal (all projects)

**Windows (PowerShell)**, from the folder where you extracted the package:

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills\tasktide" | Out-Null
New-Item -ItemType Directory -Force "$HOME\.claude\agents" | Out-Null
Copy-Item ".\.claude\skills\tasktide\*" "$HOME\.claude\skills\tasktide\" -Recurse
Copy-Item ".\.claude\agents\*.md" "$HOME\.claude\agents\"
```

**macOS / Linux:**

```bash
mkdir -p ~/.claude/skills/tasktide ~/.claude/agents
cp -R .claude/skills/tasktide/. ~/.claude/skills/tasktide/
cp .claude/agents/*.md ~/.claude/agents/
```

These commands overwrite files with the same name. If you already have an agent with one of these names, rename yours first.

### Verify

You should have:

- `~/.claude/skills/tasktide/SKILL.md`
- `~/.claude/agents/` with the seven agent files above

Start a new Claude Code session. `/agents` should list the seven agents, and `/tasktide` should appear in the command list.

## How routing works

| Level | Profile | Flow |
|---|---|---|
| **L0** trivial | Questions, docs, isolated tweaks | Main agent or `haiku-implementer` |
| **L1** common | Small feature, localized bug | `sonnet-implementer`; `sonnet-reviewer` if there is risk |
| **L2** complex | New module, several components, ambiguous requirements | Haiku maps → Opus plans → Haiku coordinates → Sonnet implements and reviews |
| **L3** critical | Security, concurrency, data migration, core architecture | Haiku maps and decomposes → Opus plans → main agent dispatches `opus-implementer` and `opus-reviewer` directly |

### Agents

| Agent | Model | Role | Writes files |
|---|---|---|---|
| `haiku-coordinator` | Haiku | Breaks a plan into tasks and dispatches specialists | No |
| `haiku-implementer` | Haiku | Docs, boilerplate, mechanical edits | Yes |
| `opus-architect` | Opus | Architecture plan for L2/L3 work | No |
| `sonnet-implementer` | Sonnet | Features and common fixes | Yes |
| `opus-implementer` | Opus | Hard or high-risk parts | Yes |
| `sonnet-reviewer` | Sonnet | Standard review | No |
| `opus-reviewer` | Opus | Review of high-risk changes | No |

## Requirements and limits

- **Nested subagents.** `haiku-coordinator` starts other subagents. This was tested on Claude Code 2.1.316. On versions without nested subagents, it returns the task board and the main agent dispatches the work itself.
- **Model aliases.** Agents use the `haiku`, `sonnet` and `opus` aliases. The exact version behind each alias depends on your account and provider. The task list in the interface shows which model each subagent used.
- **Reviewers have Bash.** They need it to read diffs and run tests. The read-only rule is an instruction in each agent's prompt, not a technical lock. Your session permissions still apply.
- **One writer at a time.** Parallel writing happens only in isolated worktrees with non-overlapping files.
- **No commit, push or deploy** without an explicit request.
- **Other orchestration layers.** If you already use another model-routing agent set, choose one per task. Your `CLAUDE.md` takes precedence over this skill.

The skill guides orchestration through instructions. It does not replace Claude Code's permissions and does not guarantee that a subagent will start in every case.

## Uninstall

Delete `~/.claude/skills/tasktide/` and the seven agent files listed above from `~/.claude/agents/`.

## License

[MIT](LICENSE)

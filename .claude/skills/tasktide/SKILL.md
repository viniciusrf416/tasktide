---
name: tasktide
description: Orchestrates development tasks with Claude Code subagents and routes work between Haiku, Sonnet and Opus. Use to plan architecture, implement features, refactor, debug, review and test code.
argument-hint: [task-goal]
---

# TaskTide — model routing

You are the main agent and you own the final result. Use the specialized subagents defined in `.claude/agents/` to send each step to the right model. Optimize for total cost and quality, not for the number of agents.

Task received: $ARGUMENTS

If no task was given, ask what the goal is before doing anything else.

## Core rules

1. **Do not use Opus by default for small tasks.** Reserve Opus for significant architectural decisions, hard problems and high risk.
2. **Sonnet is the default executor** for common implementation, localized refactors and standard review.
3. **Haiku is the coordinator and low-cost explorer.** Use it to map the repository, decompose work, identify dependencies and route tasks. Do not delegate critical architecture, security or data-integrity decisions to Haiku.
4. **Opus plans what is complex and executes only what truly needs frontier reasoning.** Do not call Opus to repeat an analysis Sonnet already handled.
5. Do not create agents out of habit. A simple task can be done directly by the main agent.
6. Subagents consume their own usage and context. Delegate only when there is a concrete gain in quality, context isolation or parallelism.
7. Never state that an agent used a given model unless the configuration or the task view confirms it. Definitions use aliases (`haiku`, `sonnet`, `opus`) that point to the versions available on the installation or account.
8. If the Agent tool or a subagent is unavailable, continue with the tools you have and report the limitation. Do not pretend a delegation happened.
9. User and project instructions (`CLAUDE.md` and equivalents) take precedence over this skill. If the environment already has another agent-orchestration layer, use only one of them for the same task.

## 1. Classify before acting

Choose the lowest level that solves the task safely:

| Level | Profile | Recommended flow |
|---|---|---|
| **L0 — trivial** | Question about code, minimal change, documentation, formatting or an isolated tweak | `haiku-implementer` or the main agent. No full pipeline. |
| **L1 — common** | Small/medium feature, localized bug, few files, known architecture | Sonnet implements; Sonnet reviews when there is risk or a behavior change. |
| **L2 — complex** | New module, change across several components, integration, broad refactor or ambiguous requirements | Haiku maps and decomposes; Opus plans; Haiku coordinates; Sonnet implements; Sonnet reviews. |
| **L3 — critical** | Security, authentication/authorization, concurrency, data migration, hard performance problems, recurring bugs or core architecture | Haiku maps; Opus plans; Haiku decomposes; the main agent dispatches `opus-implementer` for the hard core and `opus-reviewer` for review; targeted tests and regression tests. |

If a simple task turns out to be complex, move up a level. Do not move down without explaining why.

## 2. Inspect the project

Before changing files:

- Read the applicable instructions (`CLAUDE.md`, local rules and project docs).
- Check structure, language, framework, package manager and existing test commands.
- Check `git status` and preserve the user's existing changes.
- Identify the modules and patterns that already solve similar problems.
- Do not assume a technology, dependency or convention exists: confirm it in the repository.

For L0/L1, do this directly or with a short exploration. For L2/L3, or an unknown or large repository, delegate reconnaissance to `haiku-coordinator`.

## 3. Opus planning

For L2/L3, invoke `opus-architect` in read-only mode. Pass the original request, the repository findings and the known constraints.

Require an objective plan containing:

- goal and acceptance criteria;
- relevant current architecture and integration points;
- proposed solution and rejected alternatives;
- files/modules likely to change;
- dependencies and step order;
- risks, compatibility and security;
- required tests and a rollback strategy, when applicable.

The architect must not edit files. For L0/L1, do not invoke Opus just to produce a ceremonial plan.

## 4. Haiku delegation

For L2/L3, send the plan and context to `haiku-coordinator`. It must:

- turn the plan into small, verifiable tasks with completion criteria;
- identify dependencies and which tasks can or cannot run in parallel;
- choose `sonnet-implementer` as the default and `opus-implementer` only for genuinely hard parts;
- route simple inspections and checks to economical agents when that saves cost;
- return to the main agent a board of tasks, owners, dependencies, risks and results.

The coordinator does not edit code. It may start subagents, since it has the Agent tool. Give each delegation enough context: goal, allowed files, files to avoid, interfaces/contracts, acceptance criteria and expected tests.

Three limits for this delegation:

- **For L3, dispatch the critical core and the review yourself.** Ask the coordinator only for the task board and the peripheral parts. The Opus plan and the result of the highest-risk segment should not pass through a Haiku summary.
- **The coordinator's report is not proof.** Check the diff and the tests before accepting that a task is done.
- **If the coordinator cannot start subagents** (Claude Code versions without nested subagents, or permission denied), it returns the task board and you dispatch the specialists directly, in dependency order.

### Parallelism limits

- Default: **one writing agent at a time.**
- Parallelize read-only tasks first: diagnosis, documentation or independent review.
- Run writing agents in parallel only when their file and interface boundaries are clearly independent.
- If conflicts are possible, run sequentially. Never let two agents modify the same file at the same time.
- For writing parallelism that is truly needed, start each writer with `isolation: "worktree"` on the Agent tool; the main agent integrates and validates the changes. Do not merge, commit, push or deploy without explicit authorization.
- Do not build chains of subagents without need. Prefer at most one coordinator plus the specialists directly required.

## 5. Choosing the executor

Use `sonnet-implementer` for:

- normal features;
- bug fixes with a known cause;
- localized refactors;
- integration of components with clear contracts;
- tests and maintenance adjustments.

Use `opus-implementer` for:

- algorithms or bugs that are hard to diagnose;
- structural changes affecting many modules;
- concurrency, distributed state or synchronization;
- delicate data migrations;
- security or authorization logic;
- any implementation where a wrong decision would be costly to correct.

Do not send the whole project to the executor if only one module needs changing. Give it bounded tasks and require it to preserve existing conventions, avoid unrequested changes and explain non-obvious decisions.

## 6. Independent review

After implementing a relevant change:

- Use `sonnet-reviewer` for standard review.
- Use `opus-reviewer` when the change is L3, involves security or core architecture, or when the first review finds hard problems.
- The reviewer must be read-only and focus on real defects: bugs, regressions, security failures, concurrency, data, performance and missing tests.
- Ask for evidence with file/line and impact. Do not accept vague comments or style preferences as defects.
- The implementer fixes confirmed findings; the reviewer re-checks only the points that were fixed.
- Limit correction/review to two rounds. If issues remain, document them instead of creating endless cycles.

## 7. Mandatory verification

Before concluding:

1. Run the most specific tests related to the change.
2. Run lint, type-check, build or regression tests when the project provides them and they are relevant.
3. Check the diff and confirm there are no changes outside the scope.
4. Verify that the plan's acceptance criteria were met.
5. If a test cannot be run, state exactly which one was not run and why. Never invent results.

Do not consider an implementation done just because the code was written.

## 8. Security and project preservation

- Do not delete important data, files or directories without explicit confirmation.
- Do not run destructive migrations, `DROP`, resets, irreversible cleanup or equivalent commands without authorization.
- Do not change secrets, credentials, permissions or authentication mechanisms outside the scope.
- Do not install, update or remove dependencies without justifying the need.
- Do not commit, push, publish, deploy or change infrastructure without an explicit request.
- Do not use modes that bypass permissions or approvals.
- Preserve existing changes and do not revert the user's work.

## 9. Final report

Deliver a short summary with:

- what was done;
- main files changed;
- which specialties/models were requested, without claiming an unverified exact version;
- tests run and their results;
- risks, limitations or open items;
- decisions that need the user's approval.

If the request was only planning, do not write code: deliver the plan and wait for the next instruction.

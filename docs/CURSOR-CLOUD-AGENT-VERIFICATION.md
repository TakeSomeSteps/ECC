# Cursor Cloud Agent verification (this fork)

Verification run: 2026-08-31 on Cursor Cloud Agent
`bc-382b47cb-57f0-4877-adef-96e87ec23b47` against `github.com/TakeSomeSteps/ECC`
(`main` @ `ca185ef5`, ECC 2.2.1). This is an evidence note, not a product rewrite.

**Follow-up 2026-09-01:** a fresh Cloud Agent on PR #2 (`cursor/install-cursor-ecc-agents-ab54`) still could **not** spawn ECC specialists via Task. See [Follow-up: committed `.cursor/agents` do not become Task types](#follow-up-2026-09-01-committed-cursoragents-do-not-become-task-types).

## Verdict

**Not as native Cursor subagents, on this fork as committed today.**

This Cloud Agent could use ECC **skills**, **AGENTS.md**, and **CLAUDE.md**. It could
**not** spawn ECC specialists (`planner`, `code-reviewer`, `tdd-guide`, …) through
Cursor's Task tool. `.cursor/agents/` is not in the repo; the Task schema only
exposed Cursor built-ins.

Cursor's own docs say Cloud Agents *can* load committed `.cursor/agents/*.md`.
That path is unproven here because those files are install output, not source.

## What this repo ships for Cursor

| Surface | In this checkout | Count |
|---|---|---|
| Source agents | `agents/*.md` | 68 |
| Installed Cursor agents | `.cursor/agents/ecc-*.md` | **0** (not committed) |
| Cursor skills | `.cursor/skills/*/SKILL.md` | 11 |
| Codex/shared skills also present | `.agents/skills/` | 39 |
| Canonical skills | `skills/` | 286 |
| Cursor rules | `.cursor/rules/*.md` | 39, all `.md` (9 `alwaysApply`) |
| Cursor hooks | `.cursor/hooks.json` + `.cursor/hooks/*.js` | 15 events, 17 scripts |
| Cursor commands | `.cursor/commands/` | missing |
| Cursor MCP | `.cursor/mcp.json` | missing |
| Cursor environment | `.cursor/environment.json` | missing |

README still says Cursor gets **48** agents after install. The installer plans
**68** `ecc-*.md` files from `agents/`. AGENTS.md's orchestration table lists
**32** of those names.

## What this Cloud Agent actually loaded

Live session evidence (parent agent + one nested `generalPurpose` subagent):

**Loaded**

- `AGENTS.md` (always-applied workspace rule and inlined cloud instructions)
- `CLAUDE.md` (always-applied)
- `.cursor/skills/` (all 11)
- `.agents/skills/` (the committed subset)
- Cursor product skills (`env-setup`, `walkthrough-artifacts`, …)
- Task built-ins only: `generalPurpose`, `explore`, `computerUse`,
  `videoReview`, `cursor-guide`, `ci-investigator`, `best-of-n-runner`

**Not loaded as native Cursor surfaces**

- No ECC name in the Task `subagent_type` enum (`planner`, `ecc-planner`, …)
- `.cursor/rules/*.md` — Cursor docs require `.mdc` for project rules; none of
  the 39 files appeared in always-applied rules despite `alwaysApply: true`
- `.cursor/commands/` (absent)
- `~/.cursor/agents/` (absent on the VM; Cloud Agents do not see the user's
  local home)

**Hooks:** `.cursor/hooks.json` is committed, so Cloud Agents *can* run the
supported command hooks. Cursor docs say Cloud does **not** run `sessionStart`,
`sessionEnd`, MCP execution hooks, or Tab hooks. No ECC `sessionStart` memory
context appeared in this prompt. `~/.cursor/ecc` was not created.

## Installer dry-run (this VM)

Non-interactive, no user-home install:

```bash
./install.sh --profile minimal --target cursor --dry-run --json
```

Result: plan succeeds (512 operations) targeting `./.cursor/`. It would write
**68** files as `.cursor/agents/ecc-*.md`.

Apply is **not** a no-flag operation. Because this repo already contains
`.cursor/hooks.json`, apply without a hook choice throws. Use one of:

```bash
./install.sh --profile minimal --target cursor --no-hooks
./install.sh --profile minimal --target cursor --enable-hooks
```

A real apply into `/tmp/ecc-cursor-apply-*` (not `$HOME`, not this working
tree) wrote 68 agents, 121 `.mdc` rules, 94 commands, and `.cursor/mcp.json`.

Copied agent frontmatter is still Claude Code shaped (`model: opus|sonnet|haiku`,
`tools: Read, Grep, Glob`). Cursor wants `model: inherit` (or a Cursor model
id), optional `readonly` / `is_background`. Cursor docs say unknown models
fall back; that fallback was not executed in this run.

## Hypothesis check

> Cloud Agents may not auto-load project `.cursor/agents/` the same way the
> desktop IDE does.

| Claim | Evidence |
|---|---|
| This fork's Cloud Agent did not expose ECC agents as Task subagents | Confirmed: directory missing at session start; Task enum is built-ins only |
| Adding `.cursor/agents/*.md` mid-session hot-loads them | Confirmed it does **not** (wrote `ecc-cloud-probe.md` + `ecc-planner.md`; parent Task enum unchanged; nested subagent had no Task tool at all) |
| A *new* Cloud Agent would load committed `.cursor/agents/*.md` | **Not proven here.** Cursor subagent docs and Agent SDK docs say Cloud picks those files up from the cloned repo. Needs a follow-up run after the files are committed. |

## Can Guy use ECC agents from Cursor Cloud Agent?

| Kind of “use” | On this fork today |
|---|---|
| Native Task delegation to `planner` / `ecc-planner` / … | **No** |
| Follow AGENTS.md and read `agents/*.md` as playbooks | **Yes** |
| Spawn Cursor `generalPurpose` / `explore` and paste an ECC agent prompt | **Yes** (manual, not discovery) |
| Use ECC skills | **Yes** (`.cursor/skills` + `.agents/skills`) |
| Use ECC slash commands | **No** (`.cursor/commands/` not shipped) |
| Use ECC Cursor rules as `.mdc` always-apply rules | **No** (files are `.md`) |
| Use ECC agents from local Cursor IDE after `install.sh --target cursor` | Intended path; not verified in this Cloud VM |

## Practical next step

1. **Use it now without an install:** keep this Cloud Agent (or a new one on
   the same commit). Point it at `agents/<name>.md` or spawn `generalPurpose`
   with that file as the prompt. Skills already work.
2. **To get native Cursor subagents:** generate `.cursor/agents/ecc-*.md` with
   `./install.sh --profile minimal --target cursor --no-hooks` (or
   `--enable-hooks`), **commit those files**, start a **new** Cloud Agent, and
   check whether Task lists `ecc-*` / frontmatter `name` values. Optionally
   later map `model: opus|sonnet|haiku` to `inherit`.
3. **Do not expect** `~/.cursor` user agents, ECC `sessionStart` memory, or
   Tab/MCP-execution hooks on Cloud Agents.

Do not treat running the installer inside this ECC checkout as a substitute
for committing `.cursor/agents/`. Cloud Agents clone the git revision; untracked
install output dies with the VM.

## Follow-up (2026-09-01): committed `.cursor/agents` do not become Task types

Fresh Cloud Agent `bc-873ec428-9090-442e-8f67-e80b737d318a` on branch
`cursor/install-cursor-ecc-agents-ab54` @ `2f4baa6f` (PR
https://github.com/TakeSomeSteps/ECC/pull/2). This is the run the prior note
said was still needed: files committed, new session, no mid-session writes.

**Verdict: No.** Cloud Agents on this branch cannot spawn ECC specialists via
Task.

| Check | Result |
|---|---|
| Tracked `.cursor/agents/ecc-*.md` | **68**, all `model: inherit` |
| Live Task `subagent_type` enum | Cursor built-ins only (see below) |
| ECC filename (`ecc-planner`, …) in enum | **None** |
| Frontmatter `name` (`planner`, `code-reviewer`, …) in enum | **None** |
| Spawn of an ECC specialist | Not attempted (none were spawnable) |
| Installer re-run | Not done |

Exact Task enum in this session:

- `generalPurpose`
- `explore`
- `computerUse`
- `videoReview`
- `cursor-guide`
- `ci-investigator`
- `best-of-n-runner`

Skills, `AGENTS.md`, and `CLAUDE.md` still loaded. None of the 121 committed
`.cursor/rules/*.mdc` files appeared in always-applied workspace rules. No
reinstall was performed.

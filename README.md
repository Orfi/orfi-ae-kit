![orfi-ae-kit](assets/header.png)

# orfi-ae-kit

**Two sessions, one loop — you hold the pen.**

**orfi-ae-kit** packages the **Architect/Executor** two-session AI development pattern as a set of slash-command skills. It is the **optional companion** to the generic [orfi-kit](#prerequisite--orfi-kit). The pattern was originally explored as a standalone desktop app codenamed *SonOfAnton* — that app was **shelved**. orfi-ae-kit is **skills-only: there is no app**, no daemon, no background process. Everything is commands you invoke inside your AI CLI.

## Prerequisite — orfi-kit

**Install [orfi-kit](https://github.com/) first.** orfi-ae-kit is a separate add-on, **not merged** with orfi-kit. The dependency is soft — the AE commands are nearly standalone — but the Architect/Executor workflow leans on orfi-kit's generic git / test / guardrail skills to get work done. The installer only **warns** if orfi-kit is missing (it never blocks); this README is where the requirement is documented.

## The Architect/Executor pattern

You run **two context-isolated AI sessions** against one piece of work:

- The **Executor** session implements — it writes code, runs tests, does the hands-on work.
- The **Architect** session directs and verifies — it decides what to build next, writes the task brief, and critically reviews what the Executor reports back. The Architect also acts as an **interactive thinking-partner / sidekick** to the human: helping draft the prompts that go to the Executor and helping decode the Executor's questions and results.

The two sessions never share a context window. They communicate **only through file-based relay files** on disk. A **human sits at the decision points** — approving tasks, resolving blockers, and steering the loop.

## Why two sessions, not one session with a sub-agent?

A single AI session can spawn sub-agents, but that is not the same as two truly separate sessions. The distinction matters:

- **Context isolation** — each session has its own context window. Architect and Executor work independently; neither accumulates the other's file reads, test output, or diff noise. A single session doing both roles burns context rapidly and degrades quality as the window fills.
- **Independent verification** — the Architect's job is to critically review what the Executor produces. If both roles run in the same context window the "review" is just the model reading its own prior output. Separate sessions produce a genuinely independent second read.
- **Concurrent execution** — the Executor can be running tests or building while the Architect is reviewing a previous result, updating state files, or drafting the next task. Sub-agents launched from one session block or compete; separate terminals run in parallel.
- **Explicit, auditable handoff** — relay files on disk are the only channel. Every task brief and every result is a written artefact the human can read, approve, or redirect before the loop continues. Sub-agent calls are invisible to the human unless they happen to watch the transcript.
- **Human stays in the loop** — because the human physically switches terminals and sends the relay, every cycle has a natural checkpoint. Nothing proceeds without a conscious act. A sub-agent loop can run to completion without the human ever seeing an intermediate result.
- **Model flexibility** — each session can run a different model tier. The Architect can use a reasoning-heavy model for design decisions while the Executor uses a faster model for implementation. Sub-agents launched from one session are constrained by whatever that session supports.

## The relay loop

```
set-helper-files-root (one-time per repo — configure the helper-files root)
  → orient (Architect / Executor each orient their own session)
    → relay-to-executor (Architect writes task)
      → relay-read-task (Executor reads task)
        → Executor does the work
          → relay-to-architect (Executor writes result)
            → relay-read-result (Architect reviews critically)
              → repeat (Architect relays the next task)
```

## Command reference

| Command | Role |
| --- | --- |
| `/orfi-ae-kit-set-helper-files-root` | Setup: configure (create or change) the per-repo helper-files root. |
| `/orfi-ae-kit-init` | Setup: bootstrap the kit files (orientation + onboarding + session-state placeholders) under the root — create-if-absent, never overwrites. |
| `/orfi-ae-kit-orient-architect` | Orient this session as the Architect (reads orientation + session-state + onboarding). |
| `/orfi-ae-kit-orient-executor` | Orient this session as the Executor (reads executor-orientation + session-state + onboarding). |
| `/orfi-ae-kit-relay-to-executor` | Architect: write the next task to the executor relay file. |
| `/orfi-ae-kit-relay-read-task` | Executor: read the task the Architect left. |
| `/orfi-ae-kit-relay-to-architect` | Executor: write your result/report back. |
| `/orfi-ae-kit-relay-read-result` | Architect: read and critically review the Executor's result. |
| `/orfi-ae-kit-pingpong` | Architect: autonomous relay loop via Herdr — relay, nudge, wait, read, repeat until DONE. *(Installed only with Herdr support.)* |
| `/orfi-ae-kit-pingpong-stop` | Any pane: stop a running ping-pong loop at its next checkpoint. *(Installed only with Herdr support.)* |

In Claude Code these are **commands**; in GitHub Copilot CLI the equivalent **skills** are their own slash commands. Full parity: 8 Claude commands mirrored as 8 Copilot skills.

## Install

Two equivalent installers — use whichever matches your shell.

```bash
# bash (Linux / macOS / WSL)
./install.sh
```

```powershell
# PowerShell (Windows PowerShell 5+, or pwsh anywhere)
./install.ps1
```

Each launches an interactive runtime menu:

```
Install for which runtime(s)?
  1) Claude Code
  2) OpenCode
  3) GitHub Copilot CLI
Select one or more (e.g. '1', '3', or '1 2 3' / '1,2' for several).
```

Selections may be **space- or comma-separated** (`1`, `1 2`, `1,3`, `1 2 3`).

The installer then asks whether you want **Herdr support** (autonomous
architect↔executor ping-pong). Answering yes installs/updates
[Herdr](https://herdr.dev), installs its agent skill
(`npx skills add ogulcancelik/herdr --skill herdr -g`), and includes the two
ping-pong commands. Answering no installs the kit exactly as before — manual
relay mode only. Uninstall always removes the ping-pong commands if present,
but never touches Herdr itself.

### Flags

| bash | PowerShell | Meaning |
| --- | --- | --- |
| `--link` | `-Link` | symlink instead of copy (dev mode: repo edits go live) |
| `--uninstall` | `-Uninstall` | remove an existing orfi-ae-kit install |
| `--help` / `-h` | `-Help` | print usage |

### Install destinations

| Runtime | Source | Destination |
| --- | --- | --- |
| Claude Code | `claude/commands/*.md` | `~/.claude/commands/` |
| OpenCode | `claude/commands/*.md` | `~/.config/opencode/commands/` (honours `$XDG_CONFIG_HOME`) |
| GitHub Copilot CLI | `copilot/skills/<dir>/` | `~/.copilot/skills/` |

Claude and OpenCode share the same **command** source; each gets its own copy in its own commands dir. Copilot's **skills** come from a different source dir and install to their own home.

## Uninstall

```bash
./install.sh --uninstall
```

```powershell
./install.ps1 -Uninstall
```

Removes the 8 commands from the Claude/OpenCode commands dirs and the 8 skill dirs from `~/.copilot/skills`, for whichever runtimes you select.

## The helper-files root (per-repo, configurable)

The relay, orientation, onboarding, and session-state files all live under a single **helper-files root** directory. The commands no longer hard-code a path — instead each resolves the root from a small **per-repo pointer file**, so every repo keeps its own separate state and they never overwrite each other.

**Configure it once per repo** — run `/orfi-ae-kit-set-helper-files-root` (pass the path, or it will prompt you). That writes:

```
.orfi-kits/helper-files-root   one line: the absolute path to your helper-files root
```

`.orfi-kits/` is **tracked** (committed) on working branches, so the pointer propagates worktree → epic → child worktrees via the branch checkout. It is stripped from the final tree before the epic's PR merge by `/orfi-kit-cleanup-state`. Any command that needs the root reads the pointer first; if it isn't set yet, the command configures it on the spot, then continues. There is **no fallback** to any previous default path.

**Then bootstrap the kit files** — run `/orfi-ae-kit-init` once per repo. It creates the orientation, onboarding, and session-state files under the root **only if they don't already exist** (it never overwrites; if the repo is already initialized it says so and quits). The orientation files it writes are **generic/agnostic** — all project-specific rules (stack, conventions, ADR location, test commands, security gate, terminology) live in `ONBOARDING.md`, which the orientation files redirect agents to read.

The helper-files root is a general bucket, so the kit keeps its files in a dedicated **`orfi-kits/` subfolder** under the root (the pointer stores the root; the commands append `orfi-kits/`):

```
<root>\orfi-kits\relay\relay-to-executor.md     (task: Architect → Executor)
<root>\orfi-kits\relay\relay-to-architect.md    (result: Executor → Architect)
<root>\orfi-kits\architect-orientation.md       (generic: who the Architect is / how it operates)
<root>\orfi-kits\executor-orientation.md        (generic: who the Executor is / how it operates)
<root>\orfi-kits\CLAUDE-SESSION-STATE.md        (shared handoff state — Claude)
<root>\orfi-kits\COPILOT-SESSION-STATE.md       (shared handoff state — Copilot)
<root>\orfi-kits\ONBOARDING.md                  (project single source of truth, read if it exists)
```

The same pointer is shared with **orfi-kit** (which reads `CLAUDE-SESSION-STATE.md` from it via its own `/orfi-kit-set-helper-files-root` command), so the Architect/Executor handoff and orfi-kit's state commands all agree on one root per repo.

## License

MIT — see [LICENSE](LICENSE).

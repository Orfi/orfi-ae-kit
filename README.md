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
| `/orfi-ae-kit-orient-architect` | Orient this session as the Architect (reads orientation + session-state + onboarding). |
| `/orfi-ae-kit-orient-executor` | Orient this session as the Executor (reads executor-orientation + session-state + onboarding). |
| `/orfi-ae-kit-relay-to-executor` | Architect: write the next task to the executor relay file. |
| `/orfi-ae-kit-relay-read-task` | Executor: read the task the Architect left. |
| `/orfi-ae-kit-relay-to-architect` | Executor: write your result/report back. |
| `/orfi-ae-kit-relay-read-result` | Architect: read and critically review the Executor's result. |

In Claude Code these are **commands**; in GitHub Copilot CLI the equivalent **skills** are their own slash commands. Full parity: 7 Claude commands mirrored as 7 Copilot skills.

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

Removes the 7 commands from the Claude/OpenCode commands dirs and the 7 skill dirs from `~/.copilot/skills`, for whichever runtimes you select.

## The helper-files root (per-repo, configurable)

The relay, orientation, onboarding, and session-state files all live under a single **helper-files root** directory. The commands no longer hard-code a path — instead each resolves the root from a small **per-repo pointer file**, so every repo keeps its own separate state and they never overwrite each other.

**Configure it once per repo** — run `/orfi-ae-kit-set-helper-files-root` (pass the path, or it will prompt you). That writes:

```
.orfi-kits/helper-files-root   one line: the absolute path to your helper-files root
.orfi-kits/.gitignore          a single "*" so the whole folder stays untracked
```

`.orfi-kits/` is kept **untracked** by its own self-contained `.gitignore` — no edits to your repo's root `.gitignore`, no noise for people who don't use the kit. Any command that needs the root reads the pointer first; if it isn't set yet, the command configures it on the spot, then continues. There is **no fallback** to any previous default path.

Under the configured root the kit expects (created/used as the loop runs):

```
<root>\relay\relay-to-executor.md     (task: Architect → Executor)
<root>\relay\relay-to-architect.md    (result: Executor → Architect)
<root>\architect-orientation.md       (who the Architect is / how it operates)
<root>\executor-orientation.md        (who the Executor is / how it operates)
<root>\CLAUDE-SESSION-STATE.md        (shared handoff state)
<root>\ONBOARDING.md                  (epic single source of truth, read if it exists)
```

The same pointer is shared with **orfi-kit** (which reads `CLAUDE-SESSION-STATE.md` from it via its own `/orfi-kit-set-helper-files-root` command), so the Architect/Executor handoff and orfi-kit's state commands all agree on one root per repo.

## License

MIT — see [LICENSE](LICENSE).

# Configurable helper-files root (orfi-ae-kit + orfi-kit) + orient-executor

**Date:** 2026-06-24
**Status:** Approved (design)
**Repos:** `C:\code\orfi-ae-kit`, `C:\code\orfi-kit` (both updated this session)

## Problem

1. The kit's commands hard-code one user's Windows path `C:\repos\helper_files` as the
   location of the relay, orientation, onboarding, and session-state files. The kit cannot
   be used for a different repo with its own separate state — and cannot be used by anyone
   else — without hand-editing every command file. This is the known limitation already
   flagged in the README and PRD.
2. An `orfi-ae-kit-orient-executor` command exists in the global install
   (`~/.claude/commands/`) but was never added to this repo, its installer, or its Copilot
   skill set. The Executor seat has an orient command in practice but not in the kit.
3. The repo command files have drifted behind the global install: `orient-architect`,
   `relay-to-executor`, and `relay-to-architect` are older/smaller than the global copies.

## Goal

Make the helper-files root **configurable per repo** via a small untracked pointer file
read by **both kits**, add the missing **orient-executor** command (Claude + Copilot), and
bring the orfi-ae-kit repo back in sync with the improved global command bodies.

## Key decisions

- **Pointer-file design** (not install-time templating). The kit is markdown-prompt-only
  with no runtime, so configuration is a file the command bodies instruct the session to
  read. Commands stay generic — no machine-specific text baked in.
- **Per-repo, not global.** Each working repo carries its own pointer so repos never
  overwrite each other's state. A single global path is explicitly rejected.
- **One shared, kit-neutral pointer.** orfi-kit and orfi-ae-kit read the **same** root
  because they share `CLAUDE-SESSION-STATE.md` (the handoff). Two separate roots would
  split the shared state file and break the handoff.
  - When only orfi-kit is installed, the root is **partially populated** (just
    `CLAUDE-SESSION-STATE.md`). orfi-ae-kit is a superset consumer of the same root.
- **Decoupled config commands.** Each kit ships its **own** `set-helper-files-root` command
  with identical behavior writing the same pointer. Either kit can configure the path
  standalone; neither depends on the other's command existing. (orfi-kit is only a *soft*
  prerequisite — installer warns, never blocks.)
- **One shared, idempotent config routine.** The same "ensure helper-files-root" logic is
  invoked two ways: **inline (auto)** when any command needing the root finds the pointer
  missing — it creates the folder/gitignore/pointer and continues, no restart; and
  **explicitly** via `set-helper-files-root` to create the config if absent or change the
  path if present. The explicit command is the canonical entry point; the inline path calls
  the same logic. Both create-if-missing and overwrite-if-present.
- **Every consuming command checks the pointer.** Each command that needs the root reads
  `.orfi-kits/helper-files-root` from the current repo first; if missing it runs the config
  routine, then resolves its files under the path. This rule is identical across both kits —
  ae-kit's 6 relay/orient commands and orfi-kit's persist/load-state. Whichever command runs
  first in a repo self-configures; the rest just read the pointer.
- **Both repos, this session.** orfi-kit lives at `C:\code\orfi-kit` (clean, on `main`),
  alongside this repo. The pointer is shared, so doing only half would leave orfi-kit on its
  old `../helper_files` convention while ae-kit uses the pointer — they would disagree. Both
  kits are updated in one coordinated effort (two repos, two commits, two installers).
- **Untracked via a self-contained `.gitignore`.** The pointer lives in a `.orfi-kits/`
  folder whose own `.orfi-kits/.gitignore` contains `*`, ignoring the whole folder (including
  itself). No edits to the repo's root `.gitignore` — no noise for non-kit users. The config
  command also runs `git rm --cached -r .orfi-kits/` if the folder was ever tracked, so it is
  genuinely untracked, not merely ignored.
- **No fallback to `C:\repos\helper_files`.** If the pointer is set, use it. If not, prompt
  the user for a path, write the pointer, then proceed. The old literal path is never
  reintroduced as a default.

## Naming (locked)

| Thing | Value |
| --- | --- |
| Config folder | `.orfi-kits/` (in the working repo root) |
| Pointer file | `.orfi-kits/helper-files-root` — single line: absolute path to the helper-files root |
| Ignore file | `.orfi-kits/.gitignore` containing `*` |
| Command (orfi-ae-kit) | `orfi-ae-kit-set-helper-files-root` |
| Command (orfi-kit) | `orfi-kit-set-helper-files-root` |

## The helper-files root contents

Both kits resolve files relative to the configured root:

- **orfi-ae-kit reads:** `relay/relay-to-executor.md`, `relay/relay-to-architect.md`,
  `architect-orientation.md`, `executor-orientation.md`, `CLAUDE-SESSION-STATE.md`,
  `ONBOARDING.md`
- **orfi-kit reads:** `CLAUDE-SESSION-STATE.md`

## The config routine (shared shape, both kits)

Both `set-helper-files-root` commands and every inline auto-config share this behavior:

- Take the path from arguments; if none given, prompt the user for it.
- Create `.orfi-kits/` in the current repo if missing.
- Write `.orfi-kits/.gitignore` with a single line `*`.
- Write `.orfi-kits/helper-files-root` with the path (overwrite if it already exists).
- If `.orfi-kits/` was ever tracked, run `git rm --cached -r .orfi-kits/`.
- Confirm the resolved path in one line.

The pointer-read rule used by every consuming command:

- Read `.orfi-kits/helper-files-root` in the current repo.
- If missing → run the config routine above, then continue (no restart).
- Resolve the command's files under the returned path. **No fallback** to `C:\repos\helper_files`.

## Scope — orfi-ae-kit (`C:\code\orfi-ae-kit`)

1. **New config command `orfi-ae-kit-set-helper-files-root`** (Claude command + Copilot
   SKILL.md), implementing the config routine above.

2. **New command `orfi-ae-kit-orient-executor`** (Claude command + Copilot SKILL.md), synced
   from the global copy and adapted to read the pointer.

3. **Make all 6 relay/orient commands read the pointer** (per the pointer-read rule above),
   replacing the literal `C:\repos\helper_files`. The 6 commands:
   `orient-architect`, `orient-executor`, `relay-to-executor`, `relay-read-task`,
   `relay-to-architect`, `relay-read-result`.

4. **Sync the 3 drifted commands** from the global install so the repo carries the improved
   bodies:
   - `orient-architect` — adds the "read relevant ADR spec(s) in full" step and the relay
     reconciliation step.
   - `relay-to-executor` / `relay-to-architect` — add the "update the RELAY FILE STATE
     section of `CLAUDE-SESSION-STATE.md`" step (relay files have no timestamp; the state
     note is the freshness authority).

5. **Copilot parity.** Mirror the 2 new commands (`orient-executor`, `set-helper-files-root`)
   as `copilot/skills/<name>/SKILL.md`, and apply the same pointer rule + drift sync to the
   existing 4 Copilot skills.

6. **Installer (`install.sh` + `install.ps1`, both — maintenance pair).** Add
   `orfi-ae-kit-orient-executor` and `orfi-ae-kit-set-helper-files-root` to the `COMMANDS`
   and `SKILLS` arrays. No other installer behavior changes.

7. **Docs (`README.md`).** Add the 2 new commands to the command-reference table. Replace the
   "hard-coded relay paths / known limitation" section with the new per-repo pointer-config
   flow (folder, pointer file, untracked `.gitignore`, no-fallback, run set-helper-files-root
   first). Note orfi-kit ships a twin command.

## Scope — orfi-kit (`C:\code\orfi-kit`)

1. **New config command `orfi-kit-set-helper-files-root`** (Claude command + Copilot
   SKILL.md), implementing the same config routine.

2. **Make `load-state` / `persist-state` read the pointer** (Claude commands + their Copilot
   skills), replacing the relative `../helper_files` convention with the pointer-read rule.
   `git-conventions` is **not** touched — it references `helper_files/` only as worktree
   symlink examples, unrelated to the state root.

3. **Installer (`install.sh` + `install.ps1`, both).** Add `orfi-kit-set-helper-files-root`
   to the command/skill arrays.

4. **Docs (`README.md`).** Document the config command and the per-repo pointer flow.

## Out of scope

- Updating the user-authored orientation files (`architect-orientation.md`,
  `executor-orientation.md`) inside the helper-files root, which themselves hard-code the
  path internally. These are authored content, not shipped by either kit — the user edits
  them once by hand.

## Non-goals

- No runtime, daemon, hook, or app — the kit stays markdown-prompt-only.
- No moving the helper files into the working repo (rejected: risks agents deleting
  "untracked noise", and forces a cross-kit migration of hard-coded paths).

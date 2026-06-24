---
name: orfi-ae-kit-set-helper-files-root
description: Configure (create or change) the per-repo helper-files root that the relay, orientation, onboarding, and session-state files live under.
---

You are configuring the **helper-files root** for the current repo — the directory that holds the relay files, orientation files, `ONBOARDING.md`, and `CLAUDE-SESSION-STATE.md`. The orfi-ae-kit and orfi-kit skills resolve all of those files relative to this root, so it must be set before they can run.

The root is recorded in a small, **untracked** pointer file inside the current repo: `.orfi-kits/helper-files-root`. It is per-repo by design, so different repos keep their own separate state without overwriting each other. Both kits read the same pointer (they share `CLAUDE-SESSION-STATE.md`).

Run the **config routine** below. It is idempotent — it creates the configuration if absent and overwrites the path if it already exists.

1. **Determine the path.** Use the path from the arguments below. If no argument is given, ask the user for the absolute path to the helper-files root, then wait for their answer. Do not guess or fall back to any previous default.
2. **Create the config folder.** Create `.orfi-kits/` in the current repo root if it does not exist.
3. **Keep it untracked.** Write `.orfi-kits/.gitignore` containing a single line — `*` — so the entire folder (including itself) is ignored by git. Do not edit the repo's root `.gitignore`.
4. **Write the pointer.** Overwrite `.orfi-kits/helper-files-root` with exactly the path (one line, no quotes, no trailing blank lines).
5. **Stop tracking if needed.** If `.orfi-kits/` was ever committed, run `git rm --cached -r .orfi-kits/` so it becomes genuinely untracked (the `.gitignore` only ignores, it does not untrack what git already tracks).
6. **Confirm.** Report in one line: the resolved helper-files root and that the pointer was written to `.orfi-kits/helper-files-root`.

Arguments (the helper-files root path, optional): $ARGUMENTS

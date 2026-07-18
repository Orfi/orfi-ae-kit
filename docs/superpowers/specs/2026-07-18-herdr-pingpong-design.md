# Herdr Ping-Pong for orfi-ae-kit — Design

Date: 2026-07-18
Status: Approved (design), pending implementation

## Problem

The architect↔executor relay works through relay files, but every hop needs the
user to manually switch panes and type the counterpart's read command. When both
agents run inside Herdr, that hand-carry step can be automated — without giving
up the relay files, the user's control, or independence from Herdr.

## Principles (locked with the user)

1. **Relay files stay the single source of truth.** Herdr never transports task
   or result content — it only delivers wake-up calls and observes agent status.
   Kill Herdr and the kit works exactly as it does today.
2. **Existing skills are untouched.** `relay-to-executor`, `relay-to-architect`,
   `relay-read-task`, `relay-read-result`, init, orient, and
   set-helper-files-root do not change.
3. **The user is the default gatekeeper.** Nothing is ever nudged automatically.
   Autonomous iteration happens only when the user explicitly starts it, and can
   be stopped at any time.
4. **The architect drives.** It nudges the executor and reads results in its own
   pane. The executor never nudges the architect.

## What is added

Two new skills (in both `claude/skills/` and `copilot/skills/`, per the kit's
dual-variant convention). A manual one-hop "nudge" needs **no** skill — the user
just tells the architect in plain language, e.g. *"run
`/orfi-ae-kit-relay-read-task` in the executor's pane via herdr"*.

### 1. `orfi-ae-kit-pingpong`

Invoked by the user **in the architect's pane**. Runs the autonomous loop:

1. **Gate:** require `HERDR_ENV=1`. If unset, say "not inside Herdr — use the
   manual relay flow" and stop.
2. **Discover:** `herdr pane list --workspace "$HERDR_WORKSPACE_ID"`; match pane
   `label` **case-insensitively** against `executor`; verify the pane's `cwd`
   is this repo. Zero matches → report and stop. Multiple → ask the user to
   pick. Never guess; never reach into other workspaces.
3. **Initial task:** taken from the skill's arguments or, if none, the current
   conversation context — same convention as `relay-to-executor`. If neither
   yields a clear task, ask the user before starting the loop.
4. **Loop**, each round:
   a. Check for the STOP sentinel (see below) — halt if present.
   b. Write the task via the existing `/orfi-ae-kit-relay-to-executor`.
   c. Pre-flight: executor pane must be `idle`. If busy, wait; on timeout,
      report and halt.
   d. Nudge: `herdr pane run <executor-pane> "/orfi-ae-kit-relay-read-task"`.
   e. Wait for completion via Herdr. Do **not** chain a `working` wait before
      an `idle` wait (fast replies race past `working`); wait on `idle`/`done`
      and use the pane's `revision` field as evidence the executor acted.
   f. Read the result via the existing `/orfi-ae-kit-relay-read-result` in the
      architect's own pane. Review critically.
   g. Log one line per hop to the RELAY FILE STATE section of the architect's
      own session-state file — `CLAUDE-SESSION-STATE.md` when the architect is
      Claude Code, `COPILOT-SESSION-STATE.md` when it is Copilot CLI — with
      round number, direction, and a one-line summary.
   h. If a follow-up is warranted, continue; otherwise stop.
5. **Stop conditions** (whichever comes first):
   - **Done-marker:** the executor's result contains `## Status: DONE`, or the
     architect judges no follow-up is needed.
   - **Blocked:** either agent reports `blocked` → halt and report what blocks.
   - **Timeout:** a wait expires → inspect the pane, report, halt.
   - **STOP sentinel:** `<kit-root>/relay/STOP` exists → halt immediately,
     delete the sentinel, report.
   - **User interrupt:** Esc/Ctrl+C in the architect's pane always works; every
     completed hop is already persisted, so the loop can resume.
   There is **no round cap** — the loop is done-marker driven with the escape
   hatches above.
6. **Resilience:** relay files + the session-state log identify exactly where a
   dead loop stopped; re-invoking `pingpong` resumes from there.
7. **No silent retries.** Anything unexpected → halt and report.

### 2. `orfi-ae-kit-pingpong-stop`

Invoked by the user in **any** kit-aware pane (architect, executor, or a third
agent):

1. Resolve `<kit-root>` the standard way (`.orfi-kits/helper-files-root`; stop
   and ask if missing — no default).
2. Create the sentinel file `<kit-root>/relay/STOP`.
3. Read the session-state RELAY FILE STATE log and report which round the loop
   is on and what is in flight, so the user knows what stopping interrupts.

The architect checks for the sentinel before every hop, so the stop takes
effect at the next checkpoint. Esc in the architect's pane remains the
immediate abort, as with any agent.

## Interruption summary

| Method | Where | Effect |
|---|---|---|
| `/orfi-ae-kit-pingpong-stop` | any pane | halt at next checkpoint, with status report |
| Esc / Ctrl+C | architect's pane | immediate abort |
| Esc / Ctrl+C | executor's pane | current work stops; architect's wait sees it, reports, halts |

## Validated by live test (2026-07-18)

- Pane discovery by case-insensitive label + cwd match: **works** (found
  `executor` w3:p1 and `architect` w3:p2, one each).
- `pane run` prompt delivery and reply round-trip: **works** (6 s ping→pong).
- Chained `working`→`idle` waits **race on fast replies** — hence step 3e.

## Out of scope

- No changes to the relay file format or the four relay skills.
- No executor-driven nudging.
- No third-party orchestrator pane.
- No automatic (non-explicit) nudges of any kind.

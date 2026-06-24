---
name: orfi-ae-kit-relay-read-task
description: Executor role — read the task the architect left in the executor relay file.
---

You are acting as the **Executor**. Resolve the **helper-files root** first: read `.orfi-kits/helper-files-root` in the current repo. If it is missing, configure it now via `/orfi-ae-kit-set-helper-files-root` (ask the user for the path, write the pointer), then continue. There is no default path — do not fall back to any hard-coded location.

Read the file `<helper-files-root>\relay\relay-to-executor.md` in full — this is the task the architect Claude left for you.

If the file is empty or missing, say so and stop (nothing to do yet).

Otherwise: restate the task in one or two lines to confirm you understood it, then carry it out following all project rules (TDD, XML docs on touched `.cs` members, epic git policy, scope discipline). When the work is complete, write your result back with `/orfi-ae-kit-relay-to-architect` so the architect can review it.

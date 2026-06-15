---
name: orfi-ae-kit-relay-to-executor
description: Architect role — write a task to the executor relay file (overwrite).
---

You are acting as the **Architect**. Take the task/instructions from the arguments below (or, if none given, from the current conversation context) and write them as a clear, self-contained task for the **executor** Claude.

Overwrite the entire contents of `C:\repos\helper_files\relay\relay-to-executor.md` with the task. Create the file if it does not exist. Do not append — replace the whole file.

Write the task so the executor needs no other context: state the goal, the specific files/scope, any constraints, and what "done" looks like. Use this structure:

```
# Relay → Executor

## Task
<what to do>

## Scope / files
<which files or areas; what is out of scope>

## Constraints
<TDD, XML docs on touched .cs members, epic rules, etc.>

## Done when
<verifiable completion criteria>
```

After writing, confirm in one line: the file path written and a one-sentence summary of the task. Do not start doing the task yourself — you are only relaying it.

Arguments (the task to relay, optional): $ARGUMENTS

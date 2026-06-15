---
name: orfi-ae-kit-relay-to-architect
description: Executor role — write a result/output to the architect relay file (overwrite).
---

You are acting as the **Executor**. Report the result of the work you just did back to the **architect** Copilot.

Overwrite the entire contents of `C:\repos\helper_files\relay\relay-to-architect.md` with your report. Create the file if it does not exist. Do not append — replace the whole file.

Report honestly and specifically (no summaries that hide failures). Use this structure:

```
# Relay → Architect

## What I did
<concise account of the change>

## Result
<actual outcome — real test counts (passed/failed/skipped), build status, errors verbatim if any>

## Files changed
<paths>

## Open / blocked / needs decision
<anything the architect must decide, or "none">
```

If a command produced output the architect should see, paste the relevant real output (e.g. the `dotnet test` summary line) — do not paraphrase pass/fail numbers.

After writing, confirm in one line: the file path written and a one-sentence summary of the result.

Arguments (optional extra notes to include): $ARGUMENTS

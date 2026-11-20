---
name: resume-handoff
description: Pick up work from the most recent handoff document written by the handoff skill.
argument-hint: "Optional: keyword to pick a specific handoff (e.g. project name)"
disable-model-invocation: true
---

Resume work that a previous session handed off. Work through these steps in order.

## 1. Find the handoff document

Handoff documents live in the OS temp directory (`$TMPDIR`, else `/tmp`; `%TEMP%` on Windows), not the workspace. List candidates newest first:

```bash
ls -t "${TMPDIR:-/tmp}"/*handoff*.md /tmp/*handoff*.md 2>/dev/null
```

If that finds nothing, fall back to the newest Markdown files in those directories that contain a "Suggested skills" heading.

- If the user passed an argument, pick the newest candidate whose filename or content matches it.
- Otherwise take the newest. If several were written within the last day and look like different projects, list them (filename, modified time, title) and ask which one to resume.
- If none exist, say so and stop. Do not guess the state.

## 2. Read it

Read the whole document. Note its age: if it is more than a few days old, flag that the state may have moved on.

## 3. Read referenced artifacts

Read every artifact the handoff points to: files, specs, plans, ADRs, issues, PRs, commits. Use `gh` for GitHub URLs. The handoff deliberately omits detail that lives in those artifacts, so you need them for full context. Skip anything clearly irrelevant to the next step.

## 4. Invoke suggested skills

For each skill in the "Suggested skills" section, call the Skill tool if it applies to the work about to happen. If a suggested skill is not installed, tell the user (they may need to run `skill-on <name>`) and continue without it.

## 5. Confirm current state

Do not trust the handoff blindly. Check its claims against reality with cheap, read-only checks: `git status` and `git log` in the relevant repos, whether the listed files and symlinks exist, and whether branches or PRs are in the state described. Note any drift.

Then give the user a short summary:
- what the work is and where it stands
- anything that changed since the handoff
- the proposed next step

Wait for the user to confirm before you make changes.

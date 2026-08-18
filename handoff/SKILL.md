---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up. Saves to ~/.claude/handoff/<project>/ per this user's convention, never the OS temp directory or the repo itself.
argument-hint: "What will the next session be used for?"
disable-model-invocation: true
---

Write a handoff document summarising the current conversation so a fresh agent
can continue the work.

## Where it goes

Save to (`~` = your home directory — expand it for your OS):

```
~/.claude/handoff/<project>/<YYYY-MM-DD>T<HHMM>-<slug>.md
```

| OS | `~` expands to | Full example |
|----|----------------|--------------|
| Linux | `/home/<user>` | `/home/you/.claude/handoff/mtg-ai/2026-08-11T1830-custom-fields.md` |
| macOS | `/Users/<user>` | `/Users/you/.claude/handoff/mtg-ai/2026-08-11T1830-custom-fields.md` |
| Windows | `%USERPROFILE%` (`$HOME` in PowerShell) | `C:\Users\you\.claude\handoff\mtg-ai\2026-08-11T1830-custom-fields.md` |

- `<project>` is the repo's directory name (`hobbyhub`, `mtg-ai`, …). If the
  work is **not tied to a repo** — no project directory applies — ask the user
  what to name the folder rather than guessing; only fall back to `global` if
  they have no preference. One folder per project, flat files inside — **not** a
  folder per handoff.
- `<slug>` names the **topic only**. The folder already carries the project, so
  repeating it (`hobbyhub-...`) is noise.
- The timestamp prefix sorts newest-last within each project.

**Never** save into the project's own working tree, and **never** into the OS
temp directory. The upstream `mattpocock-skills:handoff` plugin hardcodes temp;
this skill exists because that default kept reasserting itself. If you ever see
a handoff land in temp, that is the plugin default winning — move it and say so.

Get the real timestamp — do not guess it from context:
- bash / Git Bash: `date +%Y-%m-%dT%H%M`
- PowerShell: `Get-Date -Format 'yyyy-MM-ddTHHmm'`

## What goes in it

- **Reference, don't restate.** Specs, plans, ADRs, issues, commits and diffs
  are already written down — link them by path or URL. A handoff that
  paraphrases eight PR bodies wastes the next session's context on what it
  could have read at source.
- **State where things actually stand**: current branch/SHA, whether the tree
  is clean, test counts you actually ran, what is deployed.
- **"What a fresh session will get wrong"** is the highest-value section. Every
  entry should come from something that actually went wrong or was actually
  surprising this session — not generic advice. Name the failure, not just the
  rule; a rule without its failure gets rationalised away at 2am.
- **Open work, grouped by what blocks it** — implementable now, blocked on a
  decision only the user can make, or parked. "Blocked" and "not started" are
  different and the next session must not confuse them.
- **A "suggested skills" section** naming the skills the next agent should
  invoke, and why.
- **Anything deliberately left uncommitted or half-done**, so it is not
  mistaken for an accident.

## Rules

- Redact secrets: API keys, passwords, tokens, credentials, personal data.
  Credentials belong in the environment or a gitignored file, never in a doc
  that gets read aloud into a fresh context.
- If the user passed arguments, treat them as a description of what the next
  session will focus on and tailor the document to that.
- Say plainly what you could **not** verify. An unverified claim presented as
  fact is worse than an admitted gap, because the next session will build on it.
- If an earlier handoff for this project is now wrong or superseded, say so at
  the top by filename. Do not leave two documents silently disagreeing.

## Resuming

There is no auto-loading. A fresh session in the project directory picks a
handoff up only when told to read its path:

> Read `~/.claude/handoff/<project>/<file>.md` and continue from there.

(Give the tool the path expanded for your OS — see the table above.)

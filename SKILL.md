---
name: project-ai-config
description: Use when setting up a new repo's Claude Code / AI config, or bringing an existing repo up to this user's standard setup — the graphify knowledge graph + hook-guard, a CLAUDE.md with the graphify/agent-skills/handoff blocks, docs/agents/{issue-tracker,domain,triage-labels}.md, CONTEXT.md, docs/adr/, and a /startup command. Triggers: "set up the project config", "apply project-ai-config", "make this project match my setup", "scaffold the AI config", "wire up graphify and the agent docs".
---

# project-ai-config

## Overview

Reproduces the user's standard per-project AI configuration in a target repo. The
setup has two layers:

- **Machine-global** (`~/.claude/`) — the plugins and global CLAUDE.md. Already
  installed on the user's machine; this skill only *verifies* it.
- **Per-project** (in the repo) — the files this skill actually creates/edits.

Template files for every per-project artifact live in `templates/` next to this
SKILL.md. They already have placeholders — copy them in and substitute.

## When to use

- A fresh repo that should get the same graphify + agent-docs + handoff wiring.
- An existing repo missing some of it (retrofit — add only what's absent).

Not for: changing the machine-global `~/.claude/` layer (that's shared across all
projects; edit it directly, don't scaffold it per-repo).

## Prerequisites: four plugins (verify, don't reinstall)

The per-project files reference these. Confirm they're enabled in
`~/.claude/settings.json` (`enabledPlugins`); if one is missing, give the user the
install command — don't install silently.

| Plugin | Role in this setup | Install |
|--------|--------------------|---------|
| graphify | knowledge graph + hook-guard | `uv tool install graphifyy` (or `pipx install graphifyy`) then `graphify install` |
| superpowers | brainstorming, TDD, subagent dev | `/plugin marketplace add anthropics/claude-plugins-official` then `/plugin install superpowers@claude-plugins-official` |
| mattpocock-skills | handoff docs, domain-modeling, triage | `/plugin install mattpocock-skills@claude-plugins-official` (same marketplace) |
| ponytail | lazy-senior-dev coding discipline | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` |

## Procedure

Work from the repo root of the target project. For a retrofit, check each file
first and only create what's missing — never clobber an existing CLAUDE.md or
CONTEXT.md; merge the blocks in instead.

1. **Gather placeholders** (see table below) — repo `<OWNER>/<REPO>` from
   `git remote -v`, `<PROJECT>` = the repo's directory name.
2. **CLAUDE.md** — copy `templates/CLAUDE.md`. If one exists, append the four
   blocks (`## graphify`, `## Agent skills`, `## Starting a new session`) that
   aren't already there.
3. **`.claude/commands/startup.md`** — copy `templates/startup.md`.
4. **`.claude/settings.json`** — copy `templates/settings.json` (graphify
   hook-guard PreToolUse hooks). If settings.json exists, merge the two hooks in.
   Adjust the `graphify.EXE` path if the user's install differs (verify with
   `where graphify` / `which graphify`).
5. **`docs/agents/`** — copy `issue-tracker.md`, `domain.md`, `triage-labels.md`
   verbatim. Edit the triage-labels table's right column if the repo's real
   labels differ.
6. **`CONTEXT.md`** — copy `templates/CONTEXT.md` skeleton. Leave the glossary
   near-empty; `/mattpocock-skills:domain-modeling` fills it lazily.
7. **`docs/adr/README.md`** — copy `templates/adr-README.md` skeleton.
8. **Build the graph** — run `graphify .` in the repo (or invoke `/graphify`).
   This generates `graphify-out/`; don't hand-create it.
9. **`.gitignore`** — decide whether `graphify-out/` is committed (this reference
   project commits it). Match the user's preference.

## Placeholders

| Placeholder | Meaning | Source |
|-------------|---------|--------|
| `<OWNER>/<REPO>` | GitHub slug for the issue tracker | `git remote -v` |
| `<PROJECT>` | handoff folder name = repo directory name | `basename $(pwd)` |
| graphify.EXE path (in settings.json) | absolute path to the graphify binary | `where graphify` |

## Verify

- `ls graphify-out/graph.json` exists → graph built.
- Run `/startup` in the target repo: it should find the graph, confirm the
  agent-skills block, and report the handoff folder — no errors.
- Trigger a Bash/Read call and confirm the graphify hook-guard fires (the
  PreToolUse nag appears) → settings.json wired correctly.
- `grep -c "## Agent skills" CLAUDE.md` returns 1.

## Notes

This is a scaffold/reference skill — its correctness test is "following it
reproduces the setup", verified by the steps above, not by pressure scenarios.
This repo ([Sheodred/project-ai-config](https://github.com/Sheodred/project-ai-config))
*is* the canonical copy — `~/.claude/skills/project-ai-config` on any machine should be
a clone of it. When the reference project's conventions change, update the templates
here and re-pull on other machines.

# project-ai-config

A small collection of [Claude Code](https://claude.com/claude-code) skills used to configure and
maintain projects. Each top-level folder is a self-contained, installable skill — clone this repo
once, then copy or symlink whichever folder(s) you need into `~/.claude/skills/`.

| Skill | What it does | License |
|---|---|---|
| [`project-ai-config/`](project-ai-config/) | Scaffolds one person's standard per-project AI config: [graphify](https://github.com/Graphify-Labs/graphify) knowledge graph + hook-guard, `CLAUDE.md` blocks, `docs/agents/*.md`, `CONTEXT.md`, `docs/adr/`, `/startup` | MIT (this repo) |
| [`handoff/`](handoff/) | Writes a session handoff document to `~/.claude/handoff/<project>/` instead of the OS temp directory, so a fresh session can pick the work up | MIT (this repo) |
| [`task-observer/`](task-observer/) | Meta-skill that watches work sessions and turns corrections/patterns into logged, reviewable skill improvements | [CC BY 4.0](task-observer/LICENSE.txt) (redistributed, unmodified — © Eoghan Henn / rebelytics.com) |

## Install

Clone the repo, then copy the skill(s) you want into your skills directory:

```bash
git clone https://github.com/Sheodred/project-ai-config.git /tmp/project-ai-config-src

# pick whichever you want:
cp -r /tmp/project-ai-config-src/project-ai-config ~/.claude/skills/
cp -r /tmp/project-ai-config-src/handoff ~/.claude/skills/
cp -r /tmp/project-ai-config-src/task-observer ~/.claude/skills/
```

Windows / PowerShell:

```powershell
git clone https://github.com/Sheodred/project-ai-config.git "$env:TEMP\project-ai-config-src"

Copy-Item -Recurse "$env:TEMP\project-ai-config-src\project-ai-config" "$HOME\.claude\skills\"
Copy-Item -Recurse "$env:TEMP\project-ai-config-src\handoff" "$HOME\.claude\skills\"
Copy-Item -Recurse "$env:TEMP\project-ai-config-src\task-observer" "$HOME\.claude\skills\"
```

No build step. Claude Code picks up any folder under `~/.claude/skills/` with a valid `SKILL.md`
frontmatter. Verify with `/project-ai-config` or `/task-observer`, or just ask Claude Code to
"set up the project config" / watch for "task-observer" triggers — see each skill's own `SKILL.md`
for exact trigger phrases.

**`task-observer` needs one extra step to be reliably active**: pair it with a structural
activation instruction in your global `CLAUDE.md` (description-matching alone isn't enforceable —
see the "Recommended activation setup" section in `task-observer/references/environments.md`).
Without it, the skill still works but only triggers opportunistically.

---

## project-ai-config

This repo *is* the skill folder for the graphify/agent-docs/handoff scaffold. See
[`project-ai-config/SKILL.md`](project-ai-config/SKILL.md) for the full procedure.

### Prerequisites: four plugins

`project-ai-config` only *wires up references* to these — it doesn't vendor or reimplement any of
them. Install what's missing; better to link to actively maintained projects than to fork what's
already there.

| Plugin | Role here | Install | Source | License |
|--------|-----------|---------|--------|---------|
| **graphify** | knowledge graph + hook-guard | `uv tool install graphifyy` (or `pipx install graphifyy`) then `graphify install` | [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) | [Apache-2.0](https://github.com/Graphify-Labs/graphify/blob/main/LICENSE) |
| **superpowers** | brainstorming, TDD, subagent-driven dev | `/plugin marketplace add anthropics/claude-plugins-official` then `/plugin install superpowers@claude-plugins-official` | [obra/superpowers](https://github.com/obra/superpowers) | [MIT](https://github.com/obra/superpowers/blob/main/LICENSE) |
| **mattpocock-skills** | handoff docs, domain-modeling, triage | `/plugin install mattpocock-skills@claude-plugins-official` (same marketplace) | [mattpocock/skills](https://github.com/mattpocock/skills) | [MIT](https://github.com/mattpocock/skills/blob/main/LICENSE) |
| **ponytail** | lazy-senior-dev coding discipline | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | [MIT](https://github.com/DietrichGebert/ponytail/blob/main/LICENSE) |

`SKILL.md` verifies these are enabled — it never installs them silently.

### Usage on a new project

1. Open the target repo in Claude Code.
2. Ask it to "set up the project config" (or run `/project-ai-config`).
3. It fills in `<OWNER>/<REPO>` from `git remote -v` and `<PROJECT>` from the directory name,
   copies the templates in, and runs `graphify .` to build the graph.

For an existing repo that already has some of this, it retrofits — only what's missing gets
added; nothing already there gets clobbered.

---

## handoff

`/handoff` writes a session handoff document so a fresh session can continue the
work without re-deriving it.

It exists because the upstream `mattpocock-skills:handoff` plugin hardcodes the
**OS temp directory**, which conflicts with the convention the other skills here
assume: one folder per project under `~/.claude/handoff/<project>/`, flat files
named `<YYYY-MM-DD>T<HHMM>-<slug>.md`. Patching the plugin's own copy does not
hold — it lives under a version-pinned cache path, so the next plugin update
silently restores the temp default. A user-level skill survives that.

`project-ai-config`'s `startup.md` template reads handoffs from the same
location, so the two are designed to be installed together. The temp fallback in
`/startup` stays as a net for older handoffs, or for a machine still running an
unpatched plugin.

The instruction text is an independent rewrite rather than a copy; the idea of a
handoff skill, and its `argument-hint`/`disable-model-invocation` frontmatter
shape, follow [mattpocock/skills](https://github.com/mattpocock/skills) (MIT).

---

## task-observer

Unmodified redistribution of ["One Skill to Rule Them All"](https://github.com/rebelytics/one-skill-to-rule-them-all)
by Eoghan Henn ([rebelytics.com](https://rebelytics.com)) — see [`task-observer/NOTICE.md`](task-observer/NOTICE.md)
for the full attribution and license note, and the upstream repo's README/USER-GUIDE for the
complete methodology writeup.

In short: it runs alongside your normal work, logs corrections/patterns/gaps as structured
observations (`skill-observations/log.md`, anchored per-project — never in an ephemeral checkout),
and periodically turns the backlog into concrete, staged skill updates you review before
installing. Nothing is ever written live without approval.

For a single local machine, the built-in **7-day in-session fallback** (offers a review at the
start of a session once the log is 7+ days old and has open entries) covers the "review
periodically" need without any extra scheduling infrastructure — see
`task-observer/references/weekly-review.md` for the full trigger logic, including why a cloud
scheduler *can't* run this review (it has no access to a local, non-git-backed observation log).

To update this copy, pull from the canonical source rather than editing it in place here.

## License

This repo's own content — `project-ai-config/`, this README, and the top-level `LICENSE` — is
released under the [MIT license](LICENSE). `task-observer/` is a separate, unmodified upstream
work under its own [CC BY 4.0](task-observer/LICENSE.txt) license (see
[`task-observer/NOTICE.md`](task-observer/NOTICE.md)); the MIT license does not apply to that
folder. The four plugins referenced by `project-ai-config` are separate projects under their own
licenses, linked in the table above — this repo only references and installs them, it doesn't
redistribute their code.

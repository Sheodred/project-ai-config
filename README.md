# project-ai-config

A [Claude Code](https://claude.com/claude-code) skill that scaffolds one person's standard
per-project AI configuration into any repo: a [graphify](https://github.com/Graphify-Labs/graphify)
knowledge graph + hook-guard, a `CLAUDE.md` wired with graphify / agent-skills / handoff blocks,
`docs/agents/*.md`, `CONTEXT.md`, `docs/adr/`, and a `/startup` command.

This repo *is* the skill folder. Clone it straight into `~/.claude/skills/` and it's installed.

## Install

```bash
git clone https://github.com/Sheodred/project-ai-config.git ~/.claude/skills/project-ai-config
```

Windows / PowerShell:

```powershell
git clone https://github.com/Sheodred/project-ai-config.git "$HOME\.claude\skills\project-ai-config"
```

No build step. Claude Code picks up any folder under `~/.claude/skills/` that has a `SKILL.md`
with the right frontmatter — this one already does.

Verify it loaded:

```
/project-ai-config
```

or ask Claude Code to "set up the project config" in a repo — see `SKILL.md` for the full
trigger phrases.

## What's in here

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill itself — overview, when to use, procedure, verification steps |
| `templates/CLAUDE.md` | The four blocks (`## graphify`, `## Agent skills`, `## Starting a new session`) appended to a project's `CLAUDE.md` |
| `templates/startup.md` | `.claude/commands/startup.md` — the session-start routine |
| `templates/settings.json` | The graphify hook-guard `PreToolUse` hooks for `.claude/settings.json` |
| `templates/issue-tracker.md`, `domain.md`, `triage-labels.md` | `docs/agents/*.md` — how agent skills should use this repo's issue tracker and domain docs |
| `templates/CONTEXT.md` | Root `CONTEXT.md` skeleton (ubiquitous-language glossary) |
| `templates/adr-README.md` | `docs/adr/README.md` skeleton (Architecture Decision Records) |

## Prerequisites: four plugins

This skill only *wires up references* to these — it doesn't vendor or reimplement any of
them. Install what's missing; better to link to actively maintained projects than to fork
what's already there.

| Plugin | Role here | Install | Source | License |
|--------|-----------|---------|--------|---------|
| **graphify** | knowledge graph + hook-guard | `uv tool install graphifyy` (or `pipx install graphifyy`) then `graphify install` | [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) | [Apache-2.0](https://github.com/Graphify-Labs/graphify/blob/main/LICENSE) |
| **superpowers** | brainstorming, TDD, subagent-driven dev | `/plugin marketplace add anthropics/claude-plugins-official` then `/plugin install superpowers@claude-plugins-official` | [obra/superpowers](https://github.com/obra/superpowers) | [MIT](https://github.com/obra/superpowers/blob/main/LICENSE) |
| **mattpocock-skills** | handoff docs, domain-modeling, triage | `/plugin install mattpocock-skills@claude-plugins-official` (same marketplace) | [mattpocock/skills](https://github.com/mattpocock/skills) | [MIT](https://github.com/mattpocock/skills/blob/main/LICENSE) |
| **ponytail** | lazy-senior-dev coding discipline | `/plugin marketplace add DietrichGebert/ponytail` then `/plugin install ponytail@ponytail` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | [MIT](https://github.com/DietrichGebert/ponytail/blob/main/LICENSE) |

`SKILL.md` verifies these are enabled — it never installs them silently.

## Usage on a new project

Read `SKILL.md` — it has the full procedure (gather placeholders, copy each template,
build the graph, verify). Short version:

1. Open the target repo in Claude Code.
2. Ask it to "set up the project config" (or run `/project-ai-config` if you alias it as
   a command).
3. It fills in `<OWNER>/<REPO>` from `git remote -v` and `<PROJECT>` from the directory
   name, copies the templates in, and runs `graphify .` to build the graph.

For an existing repo that already has some of this, it retrofits — only what's missing
gets added; nothing already there gets clobbered.

## License

This repo (the skill text and templates) is released under the [MIT license](LICENSE).
The four plugins above are separate projects under their own licenses, linked in the
table — this repo only references and installs them, it doesn't redistribute their code.

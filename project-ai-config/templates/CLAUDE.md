## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

## Agent skills

### Issue tracker

Issues and specs are tracked as GitHub issues in <OWNER>/<REPO>. See `docs/agents/issue-tracker.md`.

### Domain docs

Single-context layout — CONTEXT.md + docs/adr/ at the repo root. See `docs/agents/domain.md`.

### Triage labels

Maps the mattpocock-skills canonical triage roles to this repo's actual
issue-tracker label strings. See `docs/agents/triage-labels.md`.


## Starting a new session

Handoff documents (`/handoff`, or `/mattpocock-skills:handoff`) are never
written into this repo. They go to `C:\Users\<user>\.claude\handoff\<PROJECT>\`,
one flat file per handoff named `<YYYY-MM-DD>T<HHMM>-<slug>.md` — e.g.
`2026-08-11T1830-custom-fields.md` (the slug names the topic only; the
project is already the folder). There's no auto-loading, a fresh session
only picks one up once told to read it, which `/startup` does.

The canonical skill is `~/.claude/skills/handoff/SKILL.md`, which is
user-level and so survives plugin updates. Prefer it if it ever disagrees
with the bundled `mattpocock-skills` copy.

At the start of a new session in this repo, run `/startup` — it covers
the graphify check, the agent-skills config check, and printing the
newest handoff doc for this repo from
`C:\Users\<user>\.claude\handoff\<PROJECT>\`.

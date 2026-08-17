Run this repo's session-start routine now, in order. Don't ask for confirmation between steps — just do them and report results concisely at the end.

1. **Task observer.** Invoke the `task-observer` skill before anything else, so observations get logged while the friction happens instead of reconstructed from the transcript afterwards. This step exists because a prose instruction in `CLAUDE.md` demonstrably does not survive a long, tool-heavy session — the numbered step is the enforcement.

2. **Graphify.** Check whether `graphify-out/graph.json` exists.
   - If it exists, skip straight to using it if the conversation needs codebase Q&A later (`graphify query "<question>"`) — don't rebuild it.
   - If it doesn't exist, invoke the `graphify` skill on the repo (`/graphify`) to build it.

3. **Agent skills config.** Check whether `docs/agents/issue-tracker.md` and `docs/agents/domain.md` already exist and whether `CLAUDE.md` has an `## Agent skills` block referencing them.
   - If both exist and look current, treat setup as already done — do not attempt to invoke `/setup-matt-pocock-skills` yourself, it is restricted to explicit user invocation and will error if you try.
   - If they're missing or look stale, tell the user to run `/setup-matt-pocock-skills` themselves — don't proceed on their behalf.

4. **Handoff doc.** Handoffs live in `C:\Users\<user>\.claude\handoff\<project>\`, one flat file per handoff named `<YYYY-MM-DD>T<HHMM>-<slug>.md`. The timestamp prefix sorts newest-last within a project's folder.
   - If this session is clearly associated with this repo, take the newest file in `C:\Users\<user>\.claude\handoff\<PROJECT>\`, read it, and print the full content so both of you have it in view before continuing.
   - If there's no clear project association, list the five most recent handoffs across all project folders — timestamp, project, and slug each — and ask which to load. Don't auto-pick one.
   - If the directory is empty or missing, say so briefly and move on — don't treat it as an error.
   - Also check the OS temp directory (`C:\Users\<user>\AppData\Local\Temp\`) as a fallback. The upstream `mattpocock-skills:handoff` plugin used to hardcode that location; since 2026-08-17 the canonical `~/.claude/skills/handoff/SKILL.md` carries the convention above instead. The plugin's cached copy was patched to match, but that path is version-pinned, so a plugin update silently restores the temp default. A doc landing in temp therefore means either an older handoff or a plugin that has since updated — read it if found, say so, and move it into the project folder.

End with a short status line: what's built, what's configured, whether a handoff was found, and what it says the next task is (if any).

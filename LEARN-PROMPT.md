# LEARN-PROMPT.md

Copy everything below the line, fill in the two placeholders, paste into any AI
agent after a project session.

---

You are updating my cookbook at `<path-to>/cookbook/` (read its AGENTS.md first —
contribution rules are binding).

Project: `<project path>`
Session transcripts (if available): `<transcript dir or "none">`

Do this:

1. **Analyze.** Read the project's source files, git history (`git log -20`),
   and session transcripts if provided. Identify every lesson that satisfies ALL of:
   - It cost debugging time, caused a bug, or forced a rewrite this session.
   - It is NOT covered in official language/framework docs or common style guides.
   - It generalizes beyond this one repo (boundary behavior, silent failure,
     toolchain trap, concurrency footgun, serialization gotcha).

2. **Dedupe against existing knowledge.** Check `cookbook/<lang>/SKILL.md` for
   existing coverage. Also check these known skill collections before claiming novelty:
   `~/.claude/skills/`, `~/.claude/rules/ecc/`, `~/.gemini/config/skills/`.
   If a lesson exists there but is wrong/incomplete for modern toolchains,
   note it — don't copy it.

3. **Create or update** `cookbook/<lang-or-topic>/SKILL.md`:
   - New topic → new folder with SKILL.md following AGENTS.md format.
   - Existing → merge new lessons in, delete anything now stale, keep ≤100 lines.
   - Polyglot lesson → `cookbook/cross/SKILL.md`.

4. **Commit.** One commit per update:
   `learn(<topic>): <one-line summary of lessons added>`

5. **Report back** in ≤5 lines: what was added where, what was rejected as
   duplicate/too-specific, current line count of each touched file.

Rules you must not break: no lesson without a concrete bug behind it; nothing a
doc already teaches; nothing that only applies to this one project.

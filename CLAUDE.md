# CLAUDE.md — cookbook

Battle-tested coding lessons from real AI-agent sessions. FOSS, plain markdown,
agent-agnostic.

## What this repo is

NOT a style guide or idiom reference — official docs own that. Every lesson here
names the actual bug that taught it. If it doesn't have a scar, it doesn't belong.

## For Claude Code sessions in this repo

- Read `AGENTS.md` before touching any SKILL.md — contribution rules are binding.
- Skills live as `<topic>/SKILL.md` with frontmatter (`name`, `description`).
  To use one elsewhere: copy into `~/.claude/skills/<name>/SKILL.md` (global) or
  `<project>/.claude/skills/<name>/SKILL.md` (per-project). Auto-discovered.
- `LEARN-PROMPT.md` = the update workflow. After analyzing any project session,
  follow it: dedupe → create-or-update → commit `learn(<topic>): <summary>`.

## Hard rules

1. ≤100 lines per SKILL.md.
2. No lesson without a concrete failure behind it.
3. Nothing official docs already teach.
4. Update > duplicate; delete stale content in the same edit.
5. Polyglot topics → `cross/`.

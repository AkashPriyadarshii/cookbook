# cookbook

Battle-tested coding lessons extracted from real AI-agent sessions. Not idioms,
not style guides — the stuff that actually broke, cost debugging time, and isn't
in any official doc.

FOSS. Plain markdown. Works with any AI agent (Claude Code, Codebuff, Cursor,
Copilot, anything that reads text).

## Structure

```
cookbook/
├── LEARN-PROMPT.md   # Paste this after any project session → agent updates the cookbook
├── AGENTS.md         # Rules for contributing agents
├── rust/             # Rust boundary lessons (FFI, persistence, cross-compile)
├── kotlin/           # Android settings wiring, silent-failure UI
├── cross/            # Polyglot lessons (multi-agent git coordination)
└── <lang-or-topic>/  # Created when a session teaches something real
```

## Using a skill

Copy `rust/SKILL.md` into your project's `.claude/skills/rust-boundaries/SKILL.md`
(or `~/.claude/skills/`) — Claude Code auto-discovers it. For other agents, paste
the content or reference the path in your repo's AGENTS.md.

## Growing it

After any project session, open LEARN-PROMPT.md, copy its contents, paste into
your agent with the project path. It analyzes the session, extracts lessons, and
creates or updates the right skill file.

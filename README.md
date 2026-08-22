# cookbook — Battle-Tested Coding Lessons from Real AI-Agent Sessions

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Lessons](https://img.shields.io/badge/lessons-14-red)
![Topics](https://img.shields.io/badge/topics-rust%20%7C%20kotlin%20%7C%20cross-green)

Every lesson in this repo broke real production code first. No idioms, no
style-guide platitudes, no rehashed official docs — only failure modes that cost
actual debugging time and that documentation omits: FFI boundary crashes, silent
data loss, cross-compilation traps, concurrency footguns, serialization gotchas.

Built for AI coding agents (Claude Code, Cursor, Copilot, Windsurf, Codebuff,
Aider, Cline, Gemini CLI, OpenCode) and the humans shipping with them. Plain
markdown, FOSS, zero dependencies — drop a file into any agent's skill directory
or reference it from your repo's AGENTS.md / CLAUDE.md / .cursorrules.

## What's inside

| Topic | Lessons | What breaks without it |
|-------|---------|------------------------|
| [Rust boundaries](rust/SKILL.md) | 6 | JNI panics kill the host process · poisoned-mutex unwraps · JSON composite-key deserialization failures · Android `.so` naming mismatch · persistence stubs that lie about saving data |
| [Android/Kotlin settings wiring](kotlin/SKILL.md) | 3 | Settings screens where 17 of 20 preferences are dead — rendered but never read · destructive buttons that show success toasts and delete nothing · JVM signature clashes when retrofitting setters |
| [Multi-agent git coordination](cross/SKILL.md) | 2 | Parallel agents on one checkout stealing each other's uncommitted edits · lost handoff context between sessions |

## Why not just read the docs?

Docs teach syntax; production teaches pain. Each lesson follows one fixed
pattern: **scar** (the bug that actually happened) → **fix** (minimal working
change) → **check** (how to catch it next time). Real examples from this repo:

- `panic = "abort"` plus one `.unwrap()` on a poisoned lock in an FFI path = an
  entire Android IME crash-looping on device. The fix is three lines of poison
  recovery, and grep-able before every release.
- A "Clear clipboard history" button showing a success toast while deleting
  nothing — the worst silent failure, because it reports victory.
- Two AI agents sharing one git checkout: agent B's `git checkout` swept agent
  A's uncommitted work into B's commit. Recovery cost an hour; prevention costs
  one `git worktree add`.

## Using a skill

Copy any `<topic>/SKILL.md` into your project's `.claude/skills/<skill-name>/SKILL.md`
or globally into `~/.claude/skills/` — Claude Code auto-discovers both. For other
agents, paste the content or reference the path from AGENTS.md, CLAUDE.md,
`.cursorrules`, or `.windsurfrules`.

Current installable skills:

- **rust-boundaries** (`rust/`) — FFI panic handling, mutex poison recovery,
  persisting composite keys as JSON rows, cross-compilation artifact discipline,
  stub honesty for native bindings
- **android-settings-wiring** (`kotlin/`) — preference key → read-site tracing,
  dead-setting audits, destructive-action handler verification
- **multi-agent-git** (`cross/`) — worktree isolation protocol, handoff-log
  discipline for repos with multiple simultaneous agents

## Growing it

After any project session, open [LEARN-PROMPT.md](LEARN-PROMPT.md), copy its
contents, paste into your agent with the project path. It analyzes source and
git history, keeps only lessons that pass the bar (real bug + not in official
docs + generalizes beyond one repo), harvests battle-tested material from
installed skill collections (ECC, gstack, gemini skills), logs every accept or
reject in [LEDGER.md](LEDGER.md), and commits as `learn(<topic>): <summary>`.

## Contributing

Read [AGENTS.md](AGENTS.md) first — binding rules for human and AI contributors:
≤100 lines per skill, every lesson cites its concrete bug, nothing official docs
already teach. Update beats duplicate; stale content dies in the same edit that
adds new.

## License

MIT. Fork it, ship it, learn from our scars.

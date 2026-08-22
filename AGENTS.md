# AGENTS.md — cookbook contribution rules

For any AI agent asked to create or update a skill here.

1. **Check before writing.** Read the target skill file first. Update > duplicate.
   If the lesson exists, sharpen it; don't add a second copy.
2. **Every lesson cites its bug.** One line: what actually broke. No lesson
   without a scar. "Best practice" without a failure story belongs in a style
   guide, not here.
3. **Thin or nothing.** ≤100 lines per SKILL.md. If it exceeds that, split by
   topic (`rust/`, `rust-ffi/`) and delete overlap.
4. **Don't duplicate official docs.** Idioms, syntax, stdlib reference → skip.
   Only capture what docs get wrong or omit: boundary behavior, silent failures,
   cross-compilation traps, concurrency footguns.
5. **Delete stale content.** If a newer toolchain invalidates a lesson, remove
   it in the same edit that adds new ones. Note the version where relevant.
6. **Polyglot topics** (git, CI, JNI boundaries spanning two languages, build
   systems) go in `cross/`.
7. **Format:** frontmatter with `name` + `description` (when-to-use trigger),
   then lessons as `## N. Title` with bug → fix → check pattern.

---
name: multi-agent-git
description: >
  Multi-agent git coordination: shared working directories cause commit theft,
  worktree isolation protocol, handoff-log discipline. Use when running
  multiple AI agents (or humans+agents) on one repo across different branches.
---

# Multi-Agent Git Coordination

Lessons from a repo run by two AI agents on separate branches simultaneously.

## 1. A checkout in a shared dir steals uncommitted edits

Bug: agent B ran `git checkout <branch-B>` in the directory where agent A was
mid-edit. A's uncommitted changes rode along and were swept into B's next
commit. Recovery meant diffing B's tip against intent and porting files back
by hand.

**Fix:** one worktree per agent, created up front:

```bash
git worktree add ../<repo>-<topic> <branch>
```

Agents never `git checkout` in a directory another agent occupies, and commit
only files inside their declared territory (never `git add -A`).

**Check:** after another agent pushes, `git show <their-tip> --stat` — none of
your in-flight files should appear.

## 2. Handoff log beats memory

Parallel agents don't see each other's sessions. A root-level HANDOFF.md with
a mandatory append-before-exit entry (did / in-flight / don't-touch), newest
first, plus a territory table in AGENTS.md ("read FIRST"), prevents both
collision and duplicated work. The log entry costs 30 seconds; untangling a
stolen commit costs an hour.

## 3. Background work must die with the activity or the hand-off races the destroy

Bug: a zero-UI trampoline activity ran the strip on a raw
`Executors.newSingleThreadExecutor()` and called `startActivity()` from
`runOnUiThread` when done. Back-press mid-strip destroyed the activity under
the running executor; the UI runnable then fired `startActivity` on a dead
activity and threw `IllegalStateException` in the originating app's face. Home
press orphan-marked the completion the same way.

**Fix:** three guards, cheap and independent:
- swallow back-press (`onBackPressed` no-op) while the short strip window is
  open — the chooser after it has its own back handling;
- bail in the UI runnable (`if (isDestroyed || isFinishing) return`) before
  any Toast/startActivity;
- `shutdownNow()` the executor in `onDestroy`.

**Check:** share → press Back mid-strip → app stays alive, no crash, no
toast, no chooser; second share during a strip re-reads via `onNewIntent`.

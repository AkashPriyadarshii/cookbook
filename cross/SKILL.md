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

---
name: android-settings-wiring
description: >
  Android settings-surface audits: preferences that exist in UI but are never
  read by the consumer, buttons whose handler only shows a toast, and
  tracing every preference key to its read site. Use when auditing an
  Android app's settings screen or wiring a new preference end-to-end.
---

# Android Settings Wiring

Lessons from auditing a production Android IME where most settings were inert.

## 1. A settings screen is not a settings system

Bug: ~20 preferences rendered fine in the UI, but the keyboard consumer read
only 3 of them at startup. Seventeen toggles did literally nothing — sliders,
switches, all silently ignored. Nothing crashed; users just couldn't tell
"off" from "broken".

The gap is invisible from either end alone: the prefs XML looks done, the
consumer code looks done. Only tracing key → read-site exposes it.

**Fix:** for every `app:key` in preferences XML, grep the consumer for its
read site. Any key with zero non-UI readers is a dead setting — wire it at
one entry point (`onStartInputView` for IMEs, `onResume` for activities) or
delete it.

**Check:** script it: extract keys from XML, grep each against source minus
the settings package. Zero hits = fail.

## 2. Toast-only handlers

Bug: "Clear clipboard history" showed a success toast and deleted nothing —
the click listener ended at `Toast.makeText(...)`. User believed data was
gone; it wasn't. Worst kind of silent failure because it reports success.

**Fix:** a destructive-action handler must perform the mutation (or visibly
fail). If the handler contains no state change and no error path, it's a lie.

**Check:** grep listeners for actions named delete/clear/reset/import; each
body must touch storage or propagate an error.

## 3. Duplicate setters = JVM signature clash

Adding a manual `setLearningEnabled()` beside a `var learningEnabled`
property compiles in isolation but clashes at the platform-signature level
(`get/set` pairs generated for properties). Bit during settings rewiring;
cost a build cycle and forced direct-field assignment instead.

Docs cover the mechanism; the trap is *when* it bites: retrofitting setters
onto existing `var`s during a refactor. Prefer assigning the property
directly — the setter already exists.

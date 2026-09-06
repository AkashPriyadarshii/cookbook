---
name: android-share-intents
description: >
  Android share sheets, Intent IPC, and ContentProvider boundary traps:
  Kotlin putExtra ArrayList serialization crashes, ACTION_SEND_MULTIPLE
  clipData null fallbacks, DISPLAY_NAME path traversal in cache dirs, and
  EXTRA_EXCLUDE_COMPONENTS recursion traps. Use when building share targets,
  trampolines, or handling incoming/outgoing media intents.
---

# Android Share & Intent IPC Lessons

Lessons from building ExifDrop, an offline zero-UI share trampoline for Android.
Each broke intent delivery, dropped batches silently, or breached sandbox boundaries.

## 1. Kotlin `putExtra` marks `ArrayList<Uri>` as `Serializable` over IPC

In `ACTION_SEND_MULTIPLE`, passing `ArrayList<Uri>` via `intent.putExtra(Intent.EXTRA_STREAM, uris)`
resolves to `putExtra(String, Serializable)` in Kotlin (`ArrayList` implements `Serializable`).
Android marks the extra `Serializable`. During Binder IPC to ActivityManagerService,
Android invokes Java serialization on each `Uri` instead of parceling. Custom or
`HierarchicalUri` instances fail or crash with `NotSerializableException`.

**Fix:** call `putParcelableArrayListExtra` explicitly:

```kotlin
// WRONG: resolves to putExtra(String, Serializable)
intent.putExtra(Intent.EXTRA_STREAM, ArrayList(uris))

// CORRECT: forces Bundle.putParcelableArrayList
intent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, ArrayList(uris))
```

**Check:** `grep -rn "putExtra.*EXTRA_STREAM.*ArrayList" app/` returns zero hits.

## 2. `ACTION_SEND_MULTIPLE` fallback: `clipData == null` drops the batch

Legacy sending apps and third-party file managers frequently send `ACTION_SEND_MULTIPLE`
with `clipData == null`, placing URIs solely in `EXTRA_STREAM`. If the fallback only
calls `IntentCompat.getParcelableExtra(intent, Intent.EXTRA_STREAM, Uri::class.java)`,
it returns `null` because the payload is an `ArrayList<Uri>`. The entire batch drops silently.

**Fix:** branch on `ACTION_SEND_MULTIPLE` and call `getParcelableArrayListExtra`:

```kotlin
if (intent.action == Intent.ACTION_SEND_MULTIPLE) {
    IntentCompat.getParcelableArrayListExtra(intent, Intent.EXTRA_STREAM, Uri::class.java)
        ?.forEach { addIfClean(it) }
}
```

**Check:** test `Intent(ACTION_SEND_MULTIPLE)` with `clipData = null` and `EXTRA_STREAM = arrayListOf(u1, u2)`.

## 3. Untrusted `DISPLAY_NAME` escapes sandbox via path traversal

Querying `OpenableColumns.DISPLAY_NAME` from `contentResolver.query(uri, ...)` returns
provider-controlled strings. A hostile provider can send `../../shared_prefs/app.xml`.
Directly using `File(cacheDir, name)` allows directory traversal and private file overwrites.

**Fix:** sanitize to a bare basename: filter to `[a-zA-Z0-9 _-.]`, collapse `..`, and trim dots:

```kotlin
val stem = sourceName?.substringBeforeLast('.', "")?.take(80)
    ?.map { if (it in "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 _-.") it else '_' }
    ?.joinToString("")?.replace(Regex("\\.{2,}"), "_")?.trim('.')
    ?.takeIf { it.isNotBlank() } ?: "share_${hash}"
```

**Check:** test passing `"../../../shared_prefs/app.xml"`, `"../"`, `"...jpg"`; assert result stays in cache dir.

## 4. `EXTRA_EXCLUDE_COMPONENTS` is silently ignored on Android < 13

`Intent.EXTRA_EXCLUDE_COMPONENTS` prevents self-selection in chooser sheets, but it was
only added in API 33. On API 24–32, Android silently ignores it, causing infinite re-share loops.

**Fix:** pair `EXTRA_EXCLUDE_COMPONENTS` with an internal URI authority loop-breaker:

```kotlin
// In outgoing chooser (API 33+):
if (Build.VERSION.SDK_INT >= 33) {
    chooser.putExtra(Intent.EXTRA_EXCLUDE_COMPONENTS, arrayOf(ComponentName(packageName, TrampolineActivity::class.java.name)))
}
// In incoming receiver (API 24-36 loop-breaker):
if (uri.authority == "$packageName.fileprovider") return
```

**Check:** test trampoline with an internal FileProvider URI; verify it drops without opening chooser.

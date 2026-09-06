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
`intent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, ArrayList(uris))`
(`putExtra` resolves to `putExtra(String, Serializable)` — wrong.)

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

`OpenableColumns.DISPLAY_NAME` is provider-controlled; a hostile provider sends
`../../shared_prefs/app.xml`, and `File(cacheDir, name)` traverses out.

**Fix:** sanitize to a bare basename (allow `[a-zA-Z0-9 _-.]`, collapse `..`,
trim dots), else fall back to a hash — then any RESULT with `/` or `..` fails the check.

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

## 5. `image/*` claims what you can't strip: whitelist, or ship dirty silently

`image/*` feeds SVG/GIF/BMP/HEIC into `exifinterface`, which has no segment to
parse — the file passes through byte-identical with XML metadata intact under a
"stripped" guarantee.

**Fix:** fail closed in `prepare()` — accept only provable types:
`if (mime !in setOf("image/jpeg","image/png","image/webp","application/pdf")) return null`

**Check:** share an SVG → skipped-toast, original untouched, nothing dirty.

## 6. Object streams: `/Info N 0 R` guard passes, PDF ships dirty

ObjStm detection only refuses when NO `/Info`/`/Metadata` ref exists. Trailer
with `/Info 3 0 R` where object 3 is packed compressed inside the ObjStm (no
literal `3 0 obj`) → the regex blankers no-op, metadata ships.

**Fix:** after resolving refs, verify each has a literal object header:
`for (n in refs) if (!Regex("""(?<!\d)$n\s+0\s+obj""").containsMatchIn(s)) throw IllegalArgumentException(...)`

**Check:** `/Info 3 0 R` + obj 3 inside ObjStm → throws, never a dirty hand-off.

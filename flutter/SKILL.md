---
name: flutter-drift-android
description: >
  Flutter, Drift SQLite, and Android Gradle lessons: DSL column name collisions,
  query builder matcher collisions, legacy plugin compileSdk mismatches, and
  AGP 9+ Built-in Kotlin plugin migration. Use when writing Drift schemas,
  writing flutter_test suites, or upgrading Android build toolchains.
---

# Flutter, Drift & Android Toolchain Lessons

Lessons from building an offline-first life tracker (Flutter + Drift + Android 16).
Each one broke a real build or codegen run.

## 1. Drift column named `text` silently breaks codegen

Naming a Drift schema column `TextColumn get text => text()();` collides with
Drift's own `Table.text()` DSL builder method. `build_runner` produces an empty
`database.g.dart` stub with no compiler error, causing a cascade of missing-class
failures across the entire app.

**Fix:** name the column `note`, `content`, or `body` — never `text`:

```dart
// WRONG: collides with Table.text()
TextColumn get text => text()();

// CORRECT:
TextColumn get note => text()();
```

**Check:** `grep -rn "TextColumn get text" lib/db/` must return zero results.

## 2. Drift query builders collide with `flutter_test` and widgets

Importing `package:drift/drift.dart` in test files exports `isNull` and `isNotNull`,
which collide with `package:matcher/matcher.dart` matchers. In UI files, Drift's
`Column` and `Table` collide with Flutter layout widgets.

**Fix:** hide colliding symbols explicitly on import:

```dart
// In test files:
import 'package:drift/drift.dart' hide isNull, isNotNull;

// In widget / UI files:
import 'package:drift/drift.dart' hide Column, Table;
```

**Check:** `flutter analyze` flags `ambiguous_import` immediately on collision.

## 3. Legacy plugin hardcodes `compileSdk 34` vs AGP 36 requirement

Older plugin versions (e.g. `file_picker` ^8.x) hardcode `compileSdk 34` in their
inner `android/build.gradle`. When dependencies require API 36, Gradle's
`checkReleaseAarMetadata` fails even if the root app specifies `compileSdk = 36`.

**Fix:** bump the plugin to a federated version using `compileSdk flutter.compileSdkVersion`:

```yaml
# WRONG: hardcodes compileSdk 34 in inner gradle
file_picker: ^8.1.2

# CORRECT: federated plugin, resolves compileSdk from Flutter SDK (36)
file_picker: ^12.1.2
```

**Check:** `flutter build apk --release --target-platform android-arm64` runs `checkReleaseAarMetadata` cleanly.

## 4. AGP 9+ / Built-in Kotlin breaks legacy KGP plugin registration

AGP 9+ with `android.builtInKotlin=true` fails during `compileReleaseJavaWithJavac`
with `cannot find symbol: class FilePickerPlugin` in `GeneratedPluginRegistrant.java`
if a plugin still applies the legacy Kotlin Gradle Plugin (KGP).

**Fix:** upgrade to federated plugin packages (`android_file_picker` via `file_picker: ^12.x`)
that support Built-in Kotlin, or migrate away from plugins lacking modern AGP support.

**Check:** `GeneratedPluginRegistrant.java` compiles without missing symbol errors.

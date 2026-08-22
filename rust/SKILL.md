---
name: rust-boundaries
description: >
  Rust boundary lessons: FFI panic handling, poison-recovery locks, JSON
  serialization of composite keys, cross-compilation artifact discipline, and
  stub honesty. Use when writing JNI/FFI code, persisting engine state, or
  cross-compiling Rust for Android/embedded targets.
---

# Rust Boundary Lessons

Lessons from a production Android IME (Rust engine via JNI). Each one broke
something real.

## 1. panic=abort + FFI = host process death

With `panic = "abort"` in the release profile, any panic inside a JNI call kills
the entire host process — the IME crash-loops on device. `.unwrap()` on a mutex
lock is the classic trigger (poisoned lock → unwrap panics → dead).

**Fix:** never `.unwrap()` on locks or FFI paths. Recover from poison:

```rust
fn engine() -> MutexGuard<'static, Predictor> {
    match ENGINE.get_or_init(|| Mutex::new(Predictor::new())).lock() {
        Ok(g) => g,
        Err(poisoned) => poisoned.into_inner(), // data still valid; recover
    }
}
```

**Check:** grep `unwrap()` in every `extern` fn before release.

## 2. Global state: OnceLock<Mutex<T>>, init-once

Engine state as `static ENGINE: OnceLock<Mutex<T>>`. Init once from the host,
reuse across calls. OnceLock can't be cleared — "destroy" just drops on process
exit; don't pretend otherwise in a destroy() function.

## 3. Composite keys aren't JSON map keys

`HashMap<(String, String), V>` won't serialize to JSON ("key must be a string").
Flatten to row structs:

```rust
#[derive(Serialize, Deserialize)]
struct Row { k1: String, k2: String, v: u32 }
```

**Check (mandatory for any persistence):** roundtrip test (save → load → state
equal) + corrupt-input test (`from_json(b"{corrupt") == false`, state unchanged).

## 4. Cross-compile artifact discipline

- Targets configured in `.cargo/config.toml`, not ad-hoc env vars.
- Artifact name ≠ crate name: `libakashboard_engine.so` must be copied as
  whatever the host's `System.loadLibrary("predictor")` expects — here
  `jniLibs/arm64-v8a/libpredictor.so`.
- Verify with `ls -la` after copy; stale .so from a previous build is silent.

## 5. Stubs that lie

A native function returning hardcoded success (`nativeSaveModel → true`) makes
the caller believe persistence works. Data loss follows. Stub = return failure /
log loudly, or don't ship the binding.

## 6. Consistency tests beat per-function ceremony

For optimized algorithms (early-exit edit distance), test equivalence against
the naive version across representative inputs — catches optimization bugs that
per-function unit tests miss. Property-style check, no framework needed.

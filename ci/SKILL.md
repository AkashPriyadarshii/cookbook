---
name: ci-verify-generated-code
description: >
  CI lessons around verifying machine-generated/build outputs in workflows:
  toolchain pins smuggled inside generated crates, lint gates on generated
  code, directory args to codegen CLIs, and assert-steps that hide evidence.
  Use when a CI job builds/verifies transpiled or generated code, or when a
  green-local/red-CI loop burns runs without visible errors.
---

# Verifying Generated Code in CI

Lessons from c2proof (C→Rust via c2rust, GitHub Actions e2e). Each cost ≥1 CI
run of blind debugging.

## 1. Codegen tools smuggle toolchain pins into their output (verified: c2rust 0.20.0)

`c2rust transpile --emit-build-files` writes `rust-toolchain.toml` into the
generated crate, pinning the nightly the tool was BUILT on
(`nightly-2022-08-08`). Host-side `cargo clippy` then fails with
`'cargo-clippy' is not installed for the toolchain 'nightly-...'` — and if you
check for a file named `rust-toolchain` only, your auto-install never fires
(rustup honors both filenames; docs don't mention either).

**Fix:** detect BOTH `rust-toolchain` and `rust-toolchain.toml`, at crate root
and one level down; parse channel (bare line OR `[toolchain] channel = "..."`),
run `rustup toolchain install <ch> --profile minimal --component clippy`
(idempotent via `rustup toolchain list`). Belt-and-braces fallback: on clippy
failure, extract the channel from the error string itself — covers any pin
mechanism you didn't anticipate.

```rust
fn stderr_missing_toolchain(stderr: &str) -> Option<String> {
    stderr.lines().find_map(|l| {
        let i = l.find("not installed for the toolchain '")?;
        let rest = &l[i + "not installed for the toolchain '".len()..];
        Some(rest[..rest.find('')?].to_string())
    })
}
```

**Check:** unit test parsing both file formats; e2e log line naming the pin
path found.

## 2. Never apply your own lint gate to generated output

Running `cargo clippy -- -D warnings` on machine-transpiled code fails ~always
— mechanical output warns by design. The gate's job is compile-success;
warnings are EVIDENCE (count them into the report), not failures.

**Fix:** `cargo clippy` plain for generated code; `-D warnings` stays for
first-party source only. Capture warning count + first error lines as report
data.

**Check:** pipeline test where generated crate compiles-with-warnings is GREEN
with warnings > 0 recorded.

## 3. Codegen CLIs reject intuitive args silently (verified: c2rust 0.20.0)

`c2rust transpile <dir>` doesn't parse the directory — it emits `Could not
parse input file. Skipping /work/src; is it well-formed C?` then later `Can't
emit build files after incremental transpiler run; skipped.` Two soft-failure
messages downstream of one wrong arg; exit code still lets the container
"succeed" partially. Explicit file lists (`transpile a.c b.c -o out
--emit-build-files`) are required.

**Fix:** enumerate entry files yourself, pass explicitly, sort for
determinism. Unit-test the arg-list builder against a temp dir (headers
excluded, dotfiles excluded, flat scan).

**Check:** test asserting arg vector equals expected explicit file list.

## 4. Assert-on-artifact steps must print the artifact

A CI step doing `grep -q "✅ compiles" "$R"` hides WHY when it fails — the
evidence sat inside REPORT.md three runs in a row while we guessed.

**Fix:**

```yaml
grep -q "✅ compiles" "$R" || { cat "$R"; exit 1; }
```

Same principle in the program under test: dump failure stderr to the CI log at
the moment of detection, don't trust a downstream artifact viewer to exist.

# Ledger

One line per lesson accepted or rejected during harvest/learning. Prevents
duplicate imports and records provenance. Newest first. Append-only — a wrong
entry is corrected by a new entry, never edited.

Format: `YYYY-MM-DD | <topic> | accept|reject | one-line lesson | source`

## Entries

- 2026-08-22 | rust | accept | panic=abort + FFI unwrap kills host process; recover from mutex poison | session (akashboard IME)
- 2026-08-22 | rust | accept | composite keys aren't JSON map keys; flatten to rows + mandatory roundtrip test | session (akashboard IME)
- 2026-08-22 | rust | accept | cross-compile artifact name ≠ loadLibrary name; verify with ls after copy | session (akashboard IME)
- 2026-08-22 | rust | accept | stub returning hardcoded success lies to caller; stub must fail or not ship | session (akashboard IME)
- 2026-08-22 | kotlin | accept | settings UI without read-site = dead setting; trace key → reader, zero hits = wire or delete | session (akashboard IME)
- 2026-08-22 | kotlin | accept | toast-only handler reports success, mutates nothing; destructive handler must touch storage or error | session (akashboard IME)
- 2026-08-22 | kotlin | reject | duplicate setter JVM signature clash mechanism — official Kotlin docs teach it; kept only for when-it-bites framing in existing lesson | docs check
- 2026-08-22 | cross | accept | shared-dir checkout steals another agent's uncommitted edits; one worktree per agent, no add -A | session (multi-agent repo)
- 2026-08-22 | ci | accept | codegen output carries rust-toolchain.toml pinning builder's nightly; detect both filenames + stderr fallback | session (c2proof, 4 CI runs)
- 2026-08-22 | ci | accept | -D warnings on generated code fails by design; warnings are evidence not gate failures | session (c2proof)
- 2026-08-22 | ci | accept | c2rust dir arg soft-fails twice downstream ("is it well-formed C?"); pass explicit file lists, test arg builder | session (c2proof)
- 2026-08-22 | ci | accept | assert-on-artifact CI step must cat the artifact on failure or evidence is invisible | session (c2proof)
- 2026-08-22 | rust | reject | cargo --manifest-path puts target/ beside Cargo.toml — official cargo docs cover target-dir location | docs check

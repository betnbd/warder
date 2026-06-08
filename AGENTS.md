# AGENTS.md

## Purpose

Warder is a mothballed Linux supervised-session prototype: a CLI (`warder`) that runs an agent command under Linux enforcement (Landlock, seccomp, cgroups), journals file/network activity, and persists signed receipts. This checkout keeps only the Rust workspace and eBPF probe sources.

## Stack

Rust 2021, Cargo workspace (9 crates). Single binary `warder` from `crates/cli`. SQLite via `rusqlite` (bundled). TUI via `ratatui`/`crossterm`. Optional live eBPF via `aya`.

## Build / Run / Test

```bash
cargo build --workspace
cargo test --workspace
cargo fmt --check          # README's stated verify gate
cargo run -p warder-cli -- help
```

Live eBPF is off by default and behind a feature flag (needs `aya` + a built BPF object):

```bash
cargo build -p warder-cli --features live-ebpf
```

## Layout

- `crates/cli` — binary `warder`, arg parsing, all subcommands, TUI, demo, host probe, profile setup. `src/main.rs` is thin; logic lives in `src/lib.rs`.
- `crates/core` — shared domain types (session/snapshot/status enums).
- `crates/config` — config parsing and policy loading.
- `crates/policy` — policy helpers.
- `crates/enforcement` — Landlock / seccomp / cgroup application (`libc`).
- `crates/journal` — inotify file watcher, procfs network reader, eBPF readers; owns the `live-ebpf` feature.
- `crates/db` — receipt/journal persistence (`rusqlite`); migrations in `crates/db/migrations/`.
- `crates/snapshot` — btrfs snapshot drivers.
- `crates/daemon` — experimental daemon + host capability probe.
- `ebpf/` — standalone BPF C sources (`warder_file_access.bpf.c`, `warder_network_egress.bpf.c`); not compiled by Cargo, loaded at runtime via the `WARDER_EBPF_*` env vars.

## Gotchas

- `crates/cli` workspace member name is `warder-cli`; the produced binary is `warder`. Use `-p warder-cli` for cargo, `warder` to invoke.
- The CLI's primary command is `run` (run an agent under supervision); `start`/`stop` are an optional daemon mode. See `usage()` in `crates/cli/src/lib.rs` for the authoritative command list — it is richer than the README's `Contents`.
- eBPF probes in `ebpf/` are not built by the workspace. The `live-ebpf` feature only activates when both the feature is compiled in and `WARDER_EBPF_FILE_OBJECT` / `WARDER_EBPF_NETWORK_OBJECT` point at prebuilt objects; otherwise journaling stays inotify/procfs-based and eBPF paths return an explicit "requires live-ebpf" error.
- Enforcement (Landlock/seccomp/cgroups) is Linux-specific and degrades on unsupported kernels; `warder doctor` / `warder test-host` report host capability rather than failing hard.

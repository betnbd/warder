# Warder

Warder is an experimental Rust toolkit for supervising local AI agents on Linux.
Its command-line interface combines execution controls, file and network activity
journals, and signed session receipts to make agent runs easier to inspect.

The repository is archived and retains the core Rust workspace and optional eBPF
probe sources for exploration and further development. Enforcement depends on host
kernel capabilities; this prototype is not a complete security boundary.

The removed material included the desktop app, release packaging, CI workflows, examples, integration demos, and product/security planning docs.

## Contents

- `crates/cli`: command-line entry point and CLI tests
- `crates/core`: shared domain types
- `crates/config`: config parsing and policy loading
- `crates/enforcement`: Linux enforcement helpers
- `crates/db`: local receipt/journal persistence
- `crates/journal`: file and network journal helpers
- `crates/snapshot`: snapshot helpers
- `crates/daemon`: experimental daemon support
- `crates/policy`: policy helpers
- `ebpf`: optional eBPF probe source

## Verify

```bash
cargo fmt --check
cargo test --workspace
```

## License

MIT. See [LICENSE](LICENSE).

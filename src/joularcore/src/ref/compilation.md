# Compilation

Joular Core is written in Rust and uses Cargo. A stable Rust toolchain is the only requirement.

## Default Build

```bash
cargo build --release
```

Produces two binaries in `target/release/`:
- `joularcore` / `joularcore.exe` — command-line interface
- `joularcoregui` / `joularcoregui.exe` — graphical user interface

The default build includes virtual machine support (`vm`), the HTTP/WebSocket API (`api`), and the GUI (`gui`). SBC support is not included by default.

## SBC (Raspberry Pi) Build

```bash
cargo build --release --features sbc
```

Adds single-board computer support on top of the defaults. The `sbc` feature replaces RAPL-based CPU monitoring with polynomial regression models tuned for each supported SBC. See [Supported Platforms](../guide/supported_platforms.md) for the full list of supported boards.

## Feature Selection

Use `--no-default-features` to start from a minimal build and enable only what you need:

```bash
cargo build --release --no-default-features
```

This produces a CLI-only binary with no VM support, no API, and no GUI — useful for constrained environments where binary size matters.

### Available Features

| Feature | Default | Description |
|---------|:-------:|-------------|
| `vm`    | **on**  | Enables monitoring inside virtual machines. Joular Core reads power from a shared file written by the host. |
| `api`   | **on**  | Enables the HTTP and WebSocket API server. CSV export and ring buffer output work regardless of this feature. |
| `gui`   | **on**  | Compiles the GUI binary (`joularcoregui`). |
| `sbc`   | off     | Enables SBC support. Replaces RAPL-based monitoring with regression models for Raspberry Pi and Asus Tinker Board. |

### Mix-and-Match Examples

```bash
# CLI only — no VM, no API, no GUI
cargo build --release --no-default-features

# CLI with VM support, no GUI or API
cargo build --release --no-default-features --features vm

# CLI with VM and API, no GUI (good for headless servers)
cargo build --release --no-default-features --features vm,api

# SBC with GUI, no VM or API
cargo build --release --no-default-features --features gui,sbc

# SBC with everything
cargo build --release --features sbc
```

## Cross-Compilation

Use `cargo-make` with the targets defined in `Makefile.toml`. The targets cover all supported architectures for each OS.

To build for all supported Raspberry Pi architectures (`aarch64`, `arm`, `armv7`) at once:

```bash
cargo make build-sbc
```

With the appropriate Rust cross-compilation targets installed (via `rustup target add`), you can cross-compile from any host to any supported target.

## Release Profile

The release profile in `Cargo.toml` is configured for maximum optimization and smallest binary size:

- LTO (link-time optimization) enabled
- Single codegen unit (full cross-crate optimization)
- `panic = "abort"` (no unwinding machinery)
- Debug symbols stripped

These settings mean release builds can be slow to compile but produce fast, lean binaries.

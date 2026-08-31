# Installation

Use this procedure only for components that are missing or when the user asks
for an update. Inspect existing installations first.

## OpenBaud plugin

Verify that the `openbaud:openbaud` skill and `mcp__openbaud__*` tools are
already available. If not, install from the official marketplace:

```bash
codex plugin marketplace add Leonezz/openbaud --ref v0.1.5
codex plugin add openbaud@openbaud-marketplace
```

This skill pins OpenBaud `v0.1.5`, whose release provides runtimes for Linux
x86-64 and ARM64, macOS Intel and Apple Silicon, and Windows x86-64. Do not
change or remove the pin without verifying the replacement release's platform
assets.

Start a new Codex task from the project directory after plugin installation.
Verify installation by loading the OpenBaud skill and calling `list_ports`.

## vsp-router

Check first:

```bash
command -v vsp-router
vsp-router --version
command -v cargo
command -v git
```

When installation is needed, build the locked source from the official
repository in a temporary directory:

```bash
build_root="$(mktemp -d /tmp/vsp-router.XXXXXX)"
git clone --depth 1 https://github.com/rfdonnelly/vsp-router.git "$build_root/source"
cargo build --release --locked --manifest-path "$build_root/source/Cargo.toml"
```

Resolve the user's home directory and destination explicitly, then install the
built binary to `<home>/.local/bin/vsp-router`:

```bash
install -Dm755 "$build_root/source/target/release/vsp-router" \
  /absolute/home/.local/bin/vsp-router
```

Installing into the user's home and downloading dependencies may require
approval. Do not silently fall back to a system-wide destination. Verify:

```bash
command -v vsp-router
vsp-router --version
```

If `~/.local/bin` is not on `PATH`, use the absolute binary path or update the
user's shell configuration only when requested.

Official sources:

- OpenBaud: <https://github.com/Leonezz/openbaud>
- vsp-router: <https://github.com/rfdonnelly/vsp-router>

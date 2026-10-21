# Code Quality and CI Pipeline Guide

These contracts are a Rust workspace compiled to `wasm32-unknown-unknown` for
Soroban. This guide lists the checks CI runs so you can reproduce them locally
and avoid pipeline regressions. CI (`.github/workflows/ci.yml`) has three jobs —
**Format & lint**, **Build & test**, and **Fuzz targets** — and every one must
be green before a PR is merged.

## One-time setup

The toolchain is pinned to the version CI uses (see `rust-toolchain.toml`); with
`rustup` installed, the correct toolchain and the wasm target are selected
automatically. If you need to add the target manually:

```bash
rustup target add wasm32-unknown-unknown
rustup component add rustfmt clippy
```

## Format & lint

```bash
# Check formatting (CI runs the --check form and fails on any diff)
cargo fmt --all -- --check

# Lint; warnings are treated as errors in CI
cargo clippy --all-targets --all-features -- -D warnings
```

Run `cargo fmt --all` (without `--check`) to apply formatting before committing.

## Build & test

```bash
# Release build for the on-chain wasm target
cargo build --release --target wasm32-unknown-unknown

# Run the test suite
cargo test
```

## Fuzz targets

CI runs the fuzz targets with a bounded iteration count to stay fast on every
PR. To reproduce that locally:

```bash
FUZZ_ITERATIONS=200 cargo test --test relay_contract_fuzz -p swipely-contracts
FUZZ_ITERATIONS=200 cargo test --test threshold_window_fuzz -p swipely-contracts
FUZZ_ITERATIONS=200 cargo test --test escrow_contract_fuzz -p escrow-contract
```

Use a higher `FUZZ_ITERATIONS` for a longer local campaign; see each fuzz test
file's header for details.

## Quick pre-push check

Run the lint and test jobs together before pushing:

```bash
cargo fmt --all -- --check && \
cargo clippy --all-targets --all-features -- -D warnings && \
cargo build --release --target wasm32-unknown-unknown && \
cargo test
```

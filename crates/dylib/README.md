# gpui-kit-dylib

Forces dynamic linking of the **GPUI Kit** engine for significantly faster incremental build times.

## How it works

In large Rust applications using GUI engines (such as GPUI, WGPU, and Cosmic-Text), the linker must process hundreds of static libraries (`.rlib` files) on every incremental code change, causing sluggish linking times during rapid iteration.

`gpui-kit-dylib` compiles the entire engine and its dependencies into a single shared dynamic library (`.so` on Linux, `.dylib` on macOS, `.dll` on Windows). On subsequent compilations, the linker only needs to link against the pre-compiled shared library rather than re-linking all static libraries.

## Usage

In your application's `Cargo.toml`:

```toml
[dependencies]
gpui-kit = "0.7.0"

[features]
dev = ["gpui-kit/dynamic_linking"]
```

Run in development mode:

```bash
cargo run --features dev
```

For production/release builds, simply omit the `dev` (or `dynamic_linking`) feature:

```bash
cargo build --release
```

## Optional layers

Dynamic linking preserves Kit's feature selection. With `default-features = false`,
only GPUI and Base are included; enabling Kit's `component` or `assets` feature
also includes that layer in the shared library.

## Running and distributing binaries

Use `cargo run` or `cargo test` during development. Cargo supplies the
[dynamic library search path](https://doc.rust-lang.org/cargo/reference/environment-variables.html#dynamic-library-paths)
for processes it launches. Running the executable directly is different: the
loader must also locate the Kit dynamic library and the matching Rust toolchain's
shared libraries. Copying only the executable is not sufficient.

Cargo does **not** enable RPATH by default. Its
[`rpath` profile setting](https://doc.rust-lang.org/cargo/reference/profiles.html#rpath)
is opt-in on supported platforms, and is not a portable packaging solution.
Keep `dynamic_linking` disabled for production builds unless you deliberately
package all required shared libraries. `--release` alone does not disable features.

## Platform considerations

- **Linux and macOS**: Prefer Cargo-managed execution during development. Direct
  execution needs a suitable loader search path or an explicitly configured RPATH.
- **Windows**: Large Rust dynamic libraries can exceed the PE/COFF export-symbol
  limit. Dependency optimization may reduce the symbol count, but does not
  guarantee a successful link. If linking fails, disable `dynamic_linking`.
- **WebAssembly**: The dynamic dependency is excluded on Wasm targets.

## Measuring iteration time

Measure your application on the same machine, toolchain, profile and linker.
Use separate target directories for static and dynamic builds, warm each with
`cargo build`, then make the same small application-source edit before each timed
rebuild. Repeat and compare medians. Do not use a no-op build or `cargo check` as
a linking benchmark; neither measures an application relink. The first dynamic
build can be slower, and speedups depend on the application's dependency graph.

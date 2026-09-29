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

## Platform Considerations

- **Linux (`.so`)**: Fully supported. Cargo automatically configures `$ORIGIN/deps` in RPATH, allowing executables to find `libgpui_kit_dylib.so` out of the box.
- **macOS (`.dylib`)**: Fully supported via `@rpath`. On Apple Silicon, Cargo automatically applies ad-hoc codesigning to dynamic libraries.
- **Windows (`.dll`)**: In large projects, Windows PE/COFF files have a 16-bit export symbol limit (65,535 symbols). If you encounter `too many exported symbols`, enable optimization for dependencies in your dev profile:
  ```toml
  [profile.dev.package."*"]
  opt-level = 3
  ```
- **WebAssembly (Wasm)**: Dynamic linking is unsupported on WASM targets and is automatically disabled via `#[cfg(not(target_family = "wasm"))]`.

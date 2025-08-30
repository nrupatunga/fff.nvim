# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FFF.nvim is a fast fuzzy file picker for Neovim with a dedicated Rust backend. It consists of:

- **Rust backend** (`lua/fff/rust/lib.rs`): Native module providing file indexing, fuzzy search, and git integration
- **Lua frontend** (`lua/fff/`): Neovim UI and integration layer
- **File picker core** (`lua/fff/file_picker/`): UI components, preview handling, and image support

## Build Commands

### Development Build
```bash
cargo build --release
```

### Using Nix (alternative)
```bash
nix run .#release
```

The build produces a dynamic library (`libfff_nvim.so` on Linux, `libfff_nvim.dylib` on macOS) that gets loaded by the Lua frontend.

## Code Quality Commands

### Rust
```bash
# Run tests
cargo test

# Format code
cargo fmt

# Lint with clippy (fails on warnings)
cargo clippy -- -D warnings
```

### Lua
```bash
# Format Lua code (requires stylua)
stylua --check .
stylua .
```

## Architecture

### Rust Backend Structure
- `lib.rs`: Entry point and Lua FFI bindings
- `file_picker.rs`: Core file indexing and search logic using frizbee fuzzy search
- `frecency.rs`: File access frequency tracking with LMDB database
- `git.rs`: Git status integration using libgit2
- `background_watcher.rs`: File system change monitoring
- `score.rs`: Search result scoring algorithms

### Lua Frontend Structure  
- `fff.lua`: Main plugin entry point
- `main.lua`: Core functionality and public API
- `file_picker/init.lua`: UI components and picker logic
- `file_picker/preview.lua`: File preview with chunked loading
- `file_picker/image.lua`: Image preview support (requires terminal image support)
- `picker_ui.lua`: UI rendering and keybind handling

### Key Integration Points
- Rust functions are called from Lua via FFI bindings in `lib.rs`
- File picker state is maintained in Rust global static variables
- Frecency database persists file access patterns across sessions
- Git status is refreshed automatically via background watcher

## Toolchain Requirements

- **Rust**: Nightly toolchain (specified in `rust-toolchain.toml`)
- **Dependencies**: OpenSSL, pkg-config
- **Neovim**: 0.10.0+ required for Lua frontend

## Testing

The project currently has minimal test coverage. When adding tests:
- Rust unit tests go in the same files as the code they test
- Integration tests for the Rust backend should go in `tests/` directory
- No automated Lua testing framework is configured

## Development Workflow

1. Make Rust changes and run `cargo build --release` to rebuild the native library
2. Neovim will automatically pick up the new library on restart
3. Use `:FFFDebug on` to enable debug scoring information during development
4. Check `:FFFHealth` to verify the backend is working correctly
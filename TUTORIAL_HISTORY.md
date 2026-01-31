# Rust Tutorial History

A running log of Rust concepts learned while building this project.

---

## Concepts Covered

### Toolchain Setup (Step 1.1) - 2026-01-30
- **rustup**: Rust's toolchain manager - installs and manages Rust versions
- **cargo**: Rust's package manager and build tool (like npm for Rust)
- **Crates**: Rust's term for packages/libraries (downloaded from crates.io)
- **target/ directory**: Where Cargo stores compiled dependencies and build artifacts
- First build is slow because all dependencies compile from source; subsequent builds are fast due to caching

### Project Structure & Basic Syntax (Step 1.2) - 2026-01-30
- **Attributes** (`#[...]`): Metadata attached to items. `#![...]` applies to the whole file.
- **Conditional compilation** (`cfg_attr`): Include code only under certain conditions (e.g., release vs debug).
- **`fn main()`**: Entry point of every Rust binary.
- **`pub`**: Makes items public (visible outside their module).
- **Path separator (`::`)**: Used to access items in modules, like `tauri_temp_lib::run()`.
- **String types**:
  - `&str`: Borrowed string slice (immutable reference to string data)
  - `String`: Owned, heap-allocated, growable string
- **Macros** (identified by `!`): Like `format!()`, `println!()` - code that generates code.
- **`.expect()`**: Unwraps a `Result`, panics with message on error.
- **Cargo.toml**: Defines package metadata and dependencies (like package.json for Rust).


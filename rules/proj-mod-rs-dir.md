# proj-mod-rs-dir

> Prefer `module.rs` over `mod.rs` for multi-file modules

## Why It Matters

Rust 2018+ introduced the adjacent-file style (`user.rs` + `user/` submodules) as the idiomatic alternative to `mod.rs`. It avoids having many files all named `mod.rs` open in an editor, makes navigation easier, and is the style recommended by Clippy's `self_named_module_files` lint.

## Two Styles

### Style 1: Adjacent file (Recommended)

```
src/
├── user.rs             # Module root
├── user/
│   ├── model.rs
│   └── repository.rs
└── lib.rs
```

```rust
// src/lib.rs
mod user;  // Looks for user.rs, then user/ for submodules

// src/user.rs
mod model;
mod repository;
pub use model::User;
```

### Style 2: mod.rs (Avoid)

```
src/
├── user/
│   ├── mod.rs          # Module root
│   ├── model.rs
│   └── repository.rs
└── lib.rs
```

```rust
// src/lib.rs
mod user;  // Looks for user/mod.rs or user.rs

// src/user/mod.rs
mod model;
mod repository;
pub use model::User;
```

## When to Use Each

| Scenario | Recommendation |
|----------|----------------|
| Any module with submodules | Adjacent file (`user.rs` + `user/`) |
| Deep nesting | Adjacent file at each level |
| Library with public modules | Consistent adjacent style throughout |
| Legacy codebase | Migrate incrementally toward adjacent style |

## Adjacent File Benefits

- No more ten tabs all titled `mod.rs` in an editor
- Module interface visible without entering folder
- Matches Rust 2018+ idiomatic style and Clippy preference
- Easier to `grep` for a module by name

## mod.rs Drawbacks

- All module roots share the same filename
- Editor tabs show `mod.rs` with no disambiguation
- Clippy `self_named_module_files` flags this style

## Example: Complex Module

```
src/
├── database.rs         # Main module, re-exports
├── database/
│   ├── connection.rs   # Connection pool
│   ├── migrations.rs   # Schema migrations
│   ├── queries.rs      # Sub-module root
│   ├── queries/
│   │   ├── user.rs
│   │   └── order.rs
│   └── error.rs
└── lib.rs
```

```rust
// src/database.rs
mod connection;
mod migrations;
mod queries;
mod error;

pub use connection::Pool;
pub use error::DatabaseError;
pub use queries::{UserQueries, OrderQueries};
```

## Enforce with Clippy

```toml
# Cargo.toml or clippy.toml
[lints.clippy]
self_named_module_files = "warn"  # Enforces adjacent style (user.rs)
# mod_module_files = "warn"       # Would enforce mod.rs style - avoid
```

## See Also

- [proj-flat-small](./proj-flat-small.md) - Keep small projects flat
- [proj-mod-by-feature](./proj-mod-by-feature.md) - Feature organization
- [proj-pub-use-reexport](./proj-pub-use-reexport.md) - Re-export patterns

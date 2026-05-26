---
name: rust-skills
description: >
  Rust coding guidelines covering ownership, error handling, async patterns,
  API design, memory optimization, performance, testing, and 170+ anti-patterns.
  Apply this skill whenever writing, reviewing, or refactoring any Rust code -
  including seemingly simple tasks like adding a function, implementing a trait,
  or adding error handling. This skill is especially important when you spot
  any of these red flags: .unwrap(), .clone(), &String, &Vec<T>, std::fs in
  async code, Box<dyn Error>, or format!() in hot paths. Do not skip this skill
  just because the Rust task seems small or straightforward.
license: MIT
metadata:
  author: leonardomso
  version: "1.1.0"
  sources:
    - Rust API Guidelines
    - Rust Performance Book
    - ripgrep, tokio, serde, polars codebases
---

# Rust Best Practices

179 rules across 14 categories for idiomatic, high-performance Rust. Prioritized CRITICAL > HIGH > MEDIUM > LOW.

## Apply Rules by Task

Start here to pick the right categories for your task:

| Task | Categories to apply |
|------|---------------------|
| New function | `own-`, `err-`, `name-` |
| New struct or public API | `api-`, `type-`, `doc-` |
| Async function or task | `async-`, `own-` |
| Error handling | `err-`, `api-` |
| Memory optimization | `mem-`, `own-`, `perf-` |
| Performance tuning | `opt-`, `mem-`, `perf-` |
| Code review | `anti-`, `lint-` |
| New crate or module | `proj-`, `lint-`, `doc-` |

## Red Flags: Catch These First

When reviewing or generating code, scan for these patterns immediately - each is a likely rule violation:

| Pattern | Rule | Fix |
|---------|------|-----|
| `.unwrap()` in non-test code | `err-no-unwrap-prod` | Use `?` or `.context()` |
| `.clone()` on a reference | `own-borrow-over-clone` | Pass `&T` or `&str` instead |
| `fn f(s: &String)` | `own-slice-over-vec` | Change to `fn f(s: &str)` |
| `fn f(v: &Vec<T>)` | `own-slice-over-vec` | Change to `fn f(v: &[T])` |
| `std::fs::read` in `async fn` | `async-tokio-fs` | Use `tokio::fs::read` |
| Lock guard across `.await` | `async-no-lock-await` | Clone data out before awaiting |
| `Box<dyn std::error::Error>` | `err-custom-type` | Use `thiserror` or `anyhow` |
| `format!()` just to build a string | `mem-avoid-format` | Use string literals or `write!()` |
| `vec.push()` in a loop, no capacity | `mem-with-capacity` | Use `Vec::with_capacity(n)` |
| `map.get(k); map.insert(k, v)` | `perf-entry-api` | Use `map.entry(k).or_insert(v)` |

## Category Reference

Read `references/rules-index.md` for the full listing of all 179 rules with links to detailed examples.

Priority overview:

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Ownership & Borrowing | CRITICAL | `own-` | 12 |
| 2 | Error Handling | CRITICAL | `err-` | 12 |
| 3 | Memory Optimization | CRITICAL | `mem-` | 15 |
| 4 | API Design | HIGH | `api-` | 15 |
| 5 | Async/Await | HIGH | `async-` | 15 |
| 6 | Compiler Optimization | HIGH | `opt-` | 12 |
| 7 | Naming Conventions | MEDIUM | `name-` | 16 |
| 8 | Type Safety | MEDIUM | `type-` | 10 |
| 9 | Testing | MEDIUM | `test-` | 13 |
| 10 | Documentation | MEDIUM | `doc-` | 11 |
| 11 | Performance Patterns | MEDIUM | `perf-` | 11 |
| 12 | Project Structure | LOW | `proj-` | 11 |
| 13 | Clippy & Linting | LOW | `lint-` | 11 |
| 14 | Anti-patterns | REFERENCE | `anti-` | 15 |

For any rule ID (e.g. `err-anyhow-app`), the full example is at `rules/<rule-id>.md`.

## Recommended Cargo.toml Profiles

```toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = true

[profile.bench]
inherits = "release"
debug = true
strip = false

[profile.dev]
opt-level = 0
debug = true

[profile.dev.package."*"]
opt-level = 3  # Optimize dependencies in dev
```

## Sources

- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Rust Performance Book](https://nnethercote.github.io/perf-book/)
- [Rust Design Patterns](https://rust-unofficial.github.io/patterns/)
- Production codebases: ripgrep, tokio, serde, polars, axum, deno
- Clippy lint documentation
- Community conventions (2024-2025)

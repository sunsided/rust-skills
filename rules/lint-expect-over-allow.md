# lint-expect-over-allow

> Prefer `#[expect(lint)]` over `#[allow(lint)]` for suppressions

## Why It Matters

`#[allow(lint)]` silently suppresses a lint forever, even after the code that triggered it is removed. Stale `#[allow]` attributes accumulate and give false confidence. `#[expect(lint)]` (stable since Rust 1.81) is identical at the suppression site but emits an `unfulfilled_lint_expectations` warning if the lint never fires - catching dead suppressions automatically.

## Bad

```rust
// Stale suppression — lint was fixed months ago, nobody noticed
#[allow(clippy::too_many_arguments)]
fn simple_fn(a: u32) -> u32 {
    a + 1
}

// No explanation for why this is allowed
#[allow(clippy::unwrap_used)]
fn parse_config(s: &str) -> Config {
    serde_json::from_str(s).unwrap()
}
```

## Good

```rust
// Fails to compile if the lint stops firing
#[expect(clippy::too_many_arguments, reason = "public API, can't reduce args without breaking callers")]
fn complex_function(a: u32, b: u32, c: u32, d: u32, e: u32, f: u32, g: u32) -> u32 {
    a + b + c + d + e + f + g
}

// Forced justification; stale suppressions are caught at compile time
#[expect(clippy::unwrap_used, reason = "startup config: panic on invalid config is intentional")]
fn load_config() -> Config {
    serde_json::from_str(include_str!("../config.json")).unwrap()
}
```

## The `reason` Field

The `reason` field is optional but strongly recommended. It:
- Documents why the suppression is intentional
- Appears in the compiler warning if the suppression becomes stale
- Serves as a forcing function to justify every suppression

```rust
// Minimal — valid, but opaque
#[expect(dead_code)]
fn legacy_fn() {}

// With reason — self-documenting
#[expect(dead_code, reason = "kept for compatibility, removed in next major version")]
fn legacy_fn() {}
```

## When `#[allow]` Is Still Appropriate

Use `#[allow]` only when you explicitly DO NOT want the "lint must fire" guarantee:

- Conditional compilation: the lint fires on some targets but not others
  ```rust
  #[cfg_attr(not(target_os = "linux"), allow(dead_code))]
  fn linux_only() {}
  ```
- Generated code (`build.rs`, proc-macros) where you can't predict what fires
- `#![allow(...)]` in `lib.rs` for a crate-wide permanent exception (e.g. `missing_docs` on an internal crate)

## Workspace / Clippy Config Interaction

`#[expect]` works at item, module, and crate level (`#![expect(...)]`). Lints suppressed in `[lints]` tables in `Cargo.toml` cannot use `#[expect]` — those are config-level, not code-level.

```toml
# Cargo.toml — use allow level here, not expect
[lints.clippy]
too_many_arguments = "allow"
```

```rust
// Code-level suppression — use expect
#[expect(clippy::too_many_arguments, reason = "...")]
fn large_fn(...) {}
```

## Clippy Lint

The `unfulfilled_lint_expectations` lint is `warn` by default when using `#[expect]`. No configuration needed.

## See Also

- [lint-workspace-lints](./lint-workspace-lints.md) - Workspace-level lint configuration
- [lint-pedantic-selective](./lint-pedantic-selective.md) - Enabling pedantic lints selectively
- [lint-deny-correctness](./lint-deny-correctness.md) - Correctness lints at deny level

# async-closures

> Use `async || {}` for closures that need to await; avoid the `|| async {}` workaround

## Why It Matters

Before Rust 1.85, async closures didn't exist — the workaround was `|| async { ... }` (a sync closure returning a future). This has subtly wrong capture semantics: it captures by move into the future, not by mutable reference into the closure. Native `async || {}` (stabilized 1.85) captures correctly and enables the `AsyncFn`/`AsyncFnMut`/`AsyncFnOnce` trait bounds.

## Bad

```rust
// Workaround: sync closure returning a future
// Captures `client` by move into the future, not the closure
let fetch = || async {
    client.get("/api/data").await
};

// Can't call it twice — client was moved into the first future
fetch().await;
fetch().await;  // Error: use of moved value
```

## Good

```rust
// Native async closure (Rust 1.85+)
// Captures `client` by ref into the closure — callable multiple times
let fetch = async || {
    client.get("/api/data").await
};

fetch().await;
fetch().await;  // Fine — client borrowed, not moved
```

## Trait Bounds

Native async closures implement the new `AsyncFn` family:

```rust
use std::future::Future;

// Old way — verbose, requires naming the Future type
async fn run_twice<F, Fut>(f: F)
where
    F: Fn() -> Fut,
    Fut: Future<Output = String>,
{
    println!("{}", f().await);
    println!("{}", f().await);
}

// New way — ergonomic AsyncFn bound (Rust 1.85+)
async fn run_twice(f: impl AsyncFn() -> String) {
    println!("{}", f().await);
    println!("{}", f().await);
}
```

| Trait | Analog | Callable |
|-------|--------|----------|
| `AsyncFnOnce` | `FnOnce` | Once |
| `AsyncFnMut` | `FnMut` | Multiple times (mutably) |
| `AsyncFn` | `Fn` | Multiple times (shared ref) |

## Common Use Cases

```rust
// Stream map with async transform
use futures::StreamExt;

let results = stream
    .map(async |item| process(item).await)
    .buffered(8)
    .collect::<Vec<_>>()
    .await;

// Retry helper accepting async closure
async fn retry<T, E>(times: usize, f: impl AsyncFn() -> Result<T, E>) -> Result<T, E> {
    for _ in 0..times - 1 {
        if let Ok(v) = f().await {
            return Ok(v);
        }
    }
    f().await
}

// Spawn with captured state
let handle = tokio::spawn(async move || {
    process_batch(&items).await
}());
```

## When `|| async {}` Is Still Fine

The old form works when you explicitly want move semantics (each call gets its own owned copy):

```rust
// Each call gets its own `config` clone — intentional
let make_client = {
    let config = config.clone();
    move || async move { Client::new(&config).await }
};
```

## See Also

- [async-fn-in-trait](./async-fn-in-trait.md) - async fn in traits (Rust 1.75+)
- [async-spawn-blocking](./async-spawn-blocking.md) - Blocking work in async context
- [async-join-parallel](./async-join-parallel.md) - Running futures concurrently

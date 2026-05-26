# async-fn-in-trait

> Use native `async fn` in traits; reach for `async-trait` only for dyn dispatch

## Why It Matters

Before Rust 1.75, async methods in traits required the `async-trait` crate (a proc-macro that boxes every returned future). Since 1.75, `async fn` works natively in traits with zero overhead - no boxing, no macro. The crate is now a niche tool, not the default.

## Bad

```rust
// Pre-1.75 workaround — unnecessary on modern Rust
use async_trait::async_trait;

#[async_trait]
trait Fetcher: Send + Sync {
    async fn fetch(&self, url: &str) -> Result<Bytes, Error>;
}

#[async_trait]
impl Fetcher for HttpFetcher {
    async fn fetch(&self, url: &str) -> Result<Bytes, Error> {
        self.client.get(url).send().await?.bytes().await
    }
}
```

## Good

```rust
// Rust 1.75+ — no crate, no boxing
trait Fetcher: Send + Sync {
    async fn fetch(&self, url: &str) -> Result<Bytes, Error>;
}

impl Fetcher for HttpFetcher {
    async fn fetch(&self, url: &str) -> Result<Bytes, Error> {
        self.client.get(url).send().await?.bytes().await
    }
}
```

## The dyn Dispatch Catch

Native `async fn` in traits is NOT object-safe — `Box<dyn Fetcher>` won't compile
without help. Two options:

### Option A: explicit `-> impl Future` signature (no extra crate)

```rust
use std::future::Future;

trait Fetcher: Send + Sync {
    fn fetch(&self, url: &str) -> impl Future<Output = Result<Bytes, Error>> + Send;
}

// Box<dyn Fetcher> now works
fn make_fetcher() -> Box<dyn Fetcher> {
    Box::new(HttpFetcher::new())
}
```

### Option B: `async-trait` crate (boxing, but ergonomic for dyn-heavy APIs)

```rust
use async_trait::async_trait;

#[async_trait]
trait Fetcher: Send + Sync {
    async fn fetch(&self, url: &str) -> Result<Bytes, Error>;
}

// Box<dyn Fetcher> compiles — each call boxes the future
fn make_fetcher() -> Box<dyn Fetcher> {
    Box::new(HttpFetcher::new())
}
```

## Decision Table

| Scenario | Approach |
|----------|----------|
| Generic `<F: Fetcher>` (static dispatch) | Native `async fn` in trait |
| `Box<dyn Fetcher>` needed | `-> impl Future + Send` or `async-trait` |
| MSRV < 1.75 | `async-trait` crate required |
| Testing / mocking with mockall | Native `async fn` works (mockall 0.12+) |

## Cargo.toml

```toml
[dependencies]
# Only needed if using Box<dyn Trait> with async methods
async-trait = { version = "0.1", optional = true }
```

## See Also

- [async-tokio-runtime](./async-tokio-runtime.md) - Runtime and spawn patterns
- [async-spawn-blocking](./async-spawn-blocking.md) - CPU work in async context
- [test-mock-traits](./test-mock-traits.md) - Mocking async traits

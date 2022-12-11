### Sized

Prerequisites

- [Marker Traits](../chapter-1/marker-traits.md)
- [Auto Traits](../chapter-1/auto-traits.md)

If a type is `Sized` that means its size in bytes is known at compile-time and it's possible to put instances of the type on the stack.

Sizedness of types and its implications is a subtle yet huge topic that affects a lot of different aspects of the language. It's so important that I wrote an entire article on it called [Sizedness in Rust](https://github.com/pretzelhammer/rust-blog/blob/master/posts/sizedness-in-rust.md) which I highly recommend reading for anyone who would like to understand sizedness in-depth. I'll summarize a few key things which are relevant to this article.

1. All generic types get an implicit `Sized` bound.

```rust
fn func<T>(t: &T) {}
// example above desugared
fn func<T: Sized>(t: &T) {}
```

2. Since there's an implicit `Sized` bound on all generic types, if we want to opt-out of this implicit bound we need to use the special _"relaxed bound"_ syntax `?Sized` which currently only exists for the `Sized` trait:

```rust
// now T can be unsized
fn func<T: ?Sized>(t: &T) {}
```

3. There's an implicit `?Sized` bound on all traits.

```rust
trait Trait {}
// example above desugared
trait Trait: ?Sized {}
```

This is so that trait objects can impl the trait. Again, all of the nitty gritty details are in [Sizedness in Rust](https://github.com/pretzelhammer/rust-blog/blob/master/posts/sizedness-in-rust.md).

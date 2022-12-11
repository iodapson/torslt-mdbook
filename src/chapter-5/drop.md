#### Drop

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)

```rust
trait Drop {
    fn drop(&mut self);
}
```

If a type impls `Drop` then `drop` will be called on the type when it goes out of scope but before it's destroyed. We will rarely need to impl this for our types but a good example of where it's useful is if a type holds on to some external resources which needs to be cleaned up when the type is destroyed.

There's a `BufWriter` type in the standard library that allows us to buffer writes to `Write` types. However, what if the `BufWriter` gets destroyed before the content in its buffer has been flushed to the underlying `Write` type? Thankfully that's not possible! The `BufWriter` impls the `Drop` trait so that `flush` is always called on it whenever it goes out of scope!

```rust
impl<W: Write> Drop for BufWriter<W> {
    fn drop(&mut self) {
        self.flush_buf();
    }
}
```

Also, `Mutex`s in Rust don't have `unlock()` methods because they don't need them! Calling `lock()` on a `Mutex` returns a `MutexGuard` which automatically unlocks the `Mutex` when it goes out of scope thanks to its `Drop` impl:

```rust
impl<T: ?Sized> Drop for MutexGuard<'_, T> {
    fn drop(&mut self) {
        unsafe {
            self.lock.inner.raw_unlock();
        }
    }
}
```

In general, if you're impling an abstraction over some resource that needs to be cleaned up after use then that's a great reason to make use of the `Drop` trait.

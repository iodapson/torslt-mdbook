### ToOwned

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Default Impls](../chapter-1/default-impls.md)
- [Clone](../chapter-3/clone.md)
- [Borrow & BorrowMut](../chapter-8/borrow--borrowmut.md)

```rust
trait ToOwned {
    type Owned: Borrow<Self>;
    fn to_owned(&self) -> Self::Owned;

    // provided default impls
    fn clone_into(&self, target: &mut Self::Owned);
}
```

`ToOwned` is a more generic version of `Clone`. `Clone` allows us to take a `&T` and turn it into an `T` but `ToOwned` allows us to take a `&Borrowed` and turn it into a `Owned` where `Owned: Borrow<Borrowed>`.

In other words, we can't "clone" a `&str` into a `String`, or a `&Path` into a `PathBuf`, or an `&OsStr` into an `OsString`, since the `clone` method signature doesn't support this kind of cross-type cloning, and that's what `ToOwned` was made for.

For similar reasons as `Borrow` and `BorrowMut`, it's good to be aware of this trait and understand why it exists but it's very rare we'll ever need to impl it for any of our types.

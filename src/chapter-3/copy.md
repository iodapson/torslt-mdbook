### Copy

Prerequisites

- [Marker Traits](../chapter-1/marker-traits.md)
- [Subtraits & Supertraits](../chapter-1/subtraits--supertraits.md)
- [Derive Macros](../chapter-1/derive-macros.md)

```rust
trait Copy: Clone {}
```

We copy `Copy` types, e.g. `T` -> `T`. `Copy` promises the copy operation will be a simple bitwise copy so it will be very fast and efficient. We cannot impl `Copy` ourselves, only the compiler can provide an impl, but we can tell it to do so by using the `Copy` derive macro, together with the `Clone` derive macro since `Copy` is a subtrait of `Clone`:

```rust
#[derive(Copy, Clone)]
struct SomeType;
```

`Copy` refines `Clone`. A clone may be slow and expensive but a copy is guaranteed to be fast and cheap, so a copy is just a fast clone. If a type impls `Copy` that makes the `Clone` impl trivial:

```rust
// this is what the derive macro generates
impl<T: Copy> Clone for T {
    // the clone method becomes just a copy
    fn clone(&self) -> Self {
        *self
    }
}
```

Impling `Copy` for a type changes its behavior when it gets moved. By default all types have _move semantics_ but once a type impls `Copy` it gets _copy semantics_. To explain the difference between the two let's examine these simple scenarios:

```rust
// a "move", src: !Copy
let dest = src;
// a "copy", src: Copy
let dest = src;
```

In both cases, `dest = src` performs a simple bitwise copy of `src`'s contents and moves the result into `dest`, the only difference is that in the case of _"a move"_ the borrow checker invalidates the `src` variable and makes sure it's not used anywhere else later and in the case of _"a copy"_ `src` remains valid and usable.

In a nutshell: Copies _are_ moves. Moves _are_ copies. The only difference is how they're treated by the borrow checker.

For a more concrete example of a move, imagine `src` was a `Vec<i32>` and its contents looked something like this:

```rust
{ data: *mut [i32], length: usize, capacity: usize }
```

When we write `dest = src` we end up with:

```rust
src = { data: *mut [i32], length: usize, capacity: usize }
dest = { data: *mut [i32], length: usize, capacity: usize }
```

At this point both `src` and `dest` have aliased mutable references to the same data, which is a big no-no, so the borrow checker invalidates the `src` variable so it can't be used again without throwing a compile error.

For a more concrete example of a copy, imagine `src` was an `Option<i32>` and its contents looked something like this:

```rust
{ is_valid: bool, data: i32 }
```

Now when we write `dest = src` we end up with:

```rust
src = { is_valid: bool, data: i32 }
dest = { is_valid: bool, data: i32 }
```

These are both usable simultaneously! Hence `Option<i32>` is `Copy`.

Although `Copy` could be an auto trait the Rust language designers decided it's simpler and safer for types to explicitly opt into copy semantics rather than silently inheriting copy semantics whenever the type is eligible, as the latter can cause surprising confusing behavior which often leads to bugs.

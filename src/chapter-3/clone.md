### Clone

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Default Impls](../chapter-1/default-impls.md)
- [Derive Macros](../chapter-1/derive-macros.md)

```rust
trait Clone {
    fn clone(&self) -> Self;
    // provided default impls
    fn clone_from(&mut self, source: &Self);
}
```

We can convert immutable references of `Clone` types into owned values, i.e. `&T` -> `T`. `Clone` makes no promises about the efficiency of this conversion so it can be slow and expensive. To quickly impl `Clone` on a type we can use the derive macro:

```rust
#[derive(Clone)]
struct SomeType {
    cloneable_member1: CloneableType1,
    cloneable_member2: CloneableType2,
    // etc
}
// macro generates impl below
impl Clone for SomeType {
    fn clone(&self) -> Self {
        SomeType {
            cloneable_member1: self.cloneable_member1.clone(),
            cloneable_member2: self.cloneable_member2.clone(),
            // etc
        }
    }
}
```

`Clone` can also be useful in constructing instances of a type within a generic context. Here's a modified example from the previous section except using `Clone` instead of `Default`:

```rust
fn guarantee_length<T: Clone>(mut vec: Vec<T>, min_len: usize, fill_with: &T) -> Vec<T> {
    for _ in 0..min_len.saturating_sub(vec.len()) {
        vec.push(fill_with.clone());
    }
    vec
}
```

People also commonly use cloning as an escape hatch to avoid dealing with the borrow checker. Managing structs with references can be challenging, but we can turn the references into owned values by cloning them.

```rust
// oof, we gotta worry about lifetimes 😟
struct SomeStruct<'a> {
    data: &'a Vec<u8>,
}
// now we're on easy street 😎
struct SomeStruct {
    data: Vec<u8>,
}
```

If we're working on a program where performance is not the utmost concern then we don't need to sweat cloning data. Rust is a low-level language that exposes a lot of low-level details so it's easy to get caught up in premature optimizations instead of actually solving the problem at hand. For many programs the best order of priorities is usually to build for correctness first, elegance second, and performance third, and only focus on performance after the program has been profiled and the performance bottlenecks have been identified. This is good general advice to follow, and if it doesn't apply to your particular program then you would know.

#### Functions

A trait function is any function whose first parameter does not use the `self` keyword.

```rust
trait Default {
    // function
    fn default() -> Self;
}
```

Trait functions can be called namespaced by the trait or implementing type:

```rust
fn main() {
    let zero: i32 = Default::default();
    let zero = i32::default();
}
```

#### Methods

A trait method is any function whose first parameter uses the `self` keyword and is of type `Self`, `&Self`, `&mut Self`. The former types can also be wrapped with a `Box`, `Rc`, `Arc`, or `Pin`.

```rust
trait Trait {
    // methods
    fn takes_self(self);
    fn takes_immut_self(&self);
    fn takes_mut_self(&mut self);
    // above methods desugared
    fn takes_self(self: Self);
    fn takes_immut_self(self: &Self);
    fn takes_mut_self(self: &mut Self);
}
// example from standard library
trait ToString {
    fn to_string(&self) -> String;
}
```

Methods can be called using the dot operator on the implementing type:

```rust
fn main() {
    let five = 5.to_string();
}
```

However, similarly to functions, they can also be called namespaced by the trait or implementing type:

```rust
fn main() {
    let five = ToString::to_string(&5);
    let five = i32::to_string(&5);
}
```

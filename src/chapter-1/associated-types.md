#### Associated Types

A trait can have associated types. This is useful when we need to use some type other than `Self` within function signatures but would still like the type to be chosen by the implementer rather than being hardcoded in the trait declaration:

```rust
trait Trait {
    type AssociatedType;
    fn func(arg: Self::AssociatedType);
}
struct SomeType;
struct OtherType;
// any type implementing Trait can
// choose the type of AssociatedType
impl Trait for SomeType {
    type AssociatedType = i8; // chooses i8
    fn func(arg: Self::AssociatedType) {}
}
impl Trait for OtherType {
    type AssociatedType = u8; // chooses u8
    fn func(arg: Self::AssociatedType) {}
}
fn main() {
    SomeType::func(-1_i8); // can only call func with i8 on SomeType
    OtherType::func(1_u8); // can only call func with u8 on OtherType
}
```

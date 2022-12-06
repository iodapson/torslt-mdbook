#### Self

`Self` always refers to the implementing type.

```rust
trait Trait {
    // always returns i32
    fn returns_num() -> i32;
    // returns implementing type
    fn returns_self() -> Self;
}
struct SomeType;
struct OtherType;
impl Trait for SomeType {
    fn returns_num() -> i32 {
        5
    }
    // Self == SomeType
    fn returns_self() -> Self {
        SomeType
    }
}
impl Trait for OtherType {
    fn returns_num() -> i32 {
        6
    }
    // Self == OtherType
    fn returns_self() -> Self {
        OtherType
    }
}
```

### Auto Traits

Auto traits are traits that get automatically implemented for a type if all of its members also impl the trait. What "members" means depends on the type, for example: fields of a struct, variants of an enum, elements of an array, items of a tuple, and so on.

All auto traits are marker traits but not all marker traits are auto traits. Auto traits must be marker traits so the compiler can provide an automatic default impl for them, which would not be possible if they had any trait items.

Examples of auto traits:

```rust
// implemented for types which are safe to send between threads
unsafe auto trait Send {}
// implemented for types whose references are safe to send between threads
unsafe auto trait Sync {}
```

### Derive Macros

The standard library exports a handful of derive macros which we can use to quickly and conveniently impl a trait on a type if all of its members also impl the trait. The derive macros are named after the traits they impl:

- [Clone](../chapter-3/clone.md)
- [Copy](../chapter-3/copy.md)
- [Debug](../chapter-4/debug.md)
- [Default](../chapter-3/default)
- [Eq](../chapter-5/partialeq--eq.md)
- [Hash](../chapter-5/hash.md)
- [Ord](../chapter-5/partialord--ord.md)
- [PartialEq](../chapter-5/partialeq--eq.md)
- [PartialOrd](../chapter-5/partialord--ord.md)

Example usage:

```rust
// macro derives Copy & Clone impl for SomeType
#[derive(Copy, Clone)]
struct SomeType;
```

Note: derive macros are just procedural macros and can do anything, there's no hard rule that they must impl a trait or that they can only work if all the members of the type impl a trait, these are just the conventions followed by the derive macros in the standard library.

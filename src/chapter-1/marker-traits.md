### Marker Traits

Marker traits are traits that have no trait items. Their job is to "mark" the implementing type as having some property which is otherwise not possible to represent using the type system.

```rust
// Impling PartialEq for a type promises
// that equality for the type has these properties:
// - symmetry: a == b implies b == a, and
// - transitivity: a == b && b == c implies a == c
// But DOES NOT promise this property:
// - reflexivity: a == a
trait PartialEq {
    fn eq(&self, other: &Self) -> bool;
}
// Eq has no trait items! The eq method is already
// declared by PartialEq, but "impling" Eq
// for a type promises this additional equality property:
// - reflexivity: a == a
trait Eq: PartialEq {}
// f64 impls PartialEq but not Eq because NaN != NaN
// i32 impls PartialEq & Eq because there's no NaNs :)
```

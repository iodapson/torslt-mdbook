#### Hash

Prerequisites

- [Self](#self)
- [Methods](#methods)
- [Generic Parameters](#generic-parameters)
- [Default Impls](#default-impls)
- [Derive Macros](#derive-macros)
- [PartialEq & Eq](#partialeq--eq)

```rust
trait Hash {
    fn hash<H: Hasher>(&self, state: &mut H);
    // provided default impls
    fn hash_slice<H: Hasher>(data: &[Self], state: &mut H);
}
```

This trait is not associated with any operator, but the best time to talk about it is right after `PartialEq` & `Eq` so here it is. `Hash` types can be hashed using a `Hasher`.

```rust
use std::hash::Hasher;
use std::hash::Hash;
struct Point {
    x: i32,
    y: i32,
}
impl Hash for Point {
    fn hash<H: Hasher>(&self, hasher: &mut H) {
        hasher.write_i32(self.x);
        hasher.write_i32(self.y);
    }
}
```

There's a derive macro which generates the same impl as above:

```rust
#[derive(Hash)]
struct Point {
    x: i32,
    y: i32,
}
```

If a type impls both `Hash` and `Eq` those impls must agree with each other such that for all `a` and `b` if `a == b` then `a.hash() == b.hash()`. So we should always use the derive macro to impl both or manually impl both, but not mix the two, otherwise we risk breaking the above invariant.

The main benefit of impling `Eq` and `Hash` for a type is that it allows us to store that type as keys in `HashMap`s and `HashSet`s.

```rust
use std::collections::HashSet;
// now our type can be stored
// in HashSets and HashMaps!
#[derive(PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}
fn example_hashset() {
    let mut points = HashSet::new();
    points.insert(Point { x: 0, y: 0 }); // ✅
}
```

#### PartialOrd & Ord

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Generic Parameters](../chapter-1/generic-parameters.md)
- [Default Impls](../chapter-1/default-impls.md)
- [Subtraits & Supertraits](../chapter-1/subtraits--supertraits.md)
- [Derive Macros](../chapter-1/derive-macros.md)
- [Sized](../chapter-2/sized.md)
- [PartialEq & Eq](./partialeq--eq.md)

```rust
enum Ordering {
    Less,
    Equal,
    Greater,
}
trait PartialOrd<Rhs = Self>: PartialEq<Rhs>
where
    Rhs: ?Sized,
{
    fn partial_cmp(&self, other: &Rhs) -> Option<Ordering>;
    // provided default impls
    fn lt(&self, other: &Rhs) -> bool;
    fn le(&self, other: &Rhs) -> bool;
    fn gt(&self, other: &Rhs) -> bool;
    fn ge(&self, other: &Rhs) -> bool;
}
```

`PartialOrd<Rhs>` types can be compared to `Rhs` types using the `<`, `<=`, `>`, and `>=` operators.

All `PartialOrd` impls must ensure that comparisons are asymmetric and transitive. That means for all `a`, `b`, and `c`:

- `a < b` implies `!(a > b)` (asymmetry)
- `a < b && b < c` implies `a < c` (transitivity)

`PartialOrd` is a subtrait of `PartialEq` and their impls must always agree with each other.

```rust
fn must_always_agree<T: PartialOrd + PartialEq>(t1: T, t2: T) {
    assert_eq!(t1.partial_cmp(&t2) == Some(Ordering::Equal), t1 == t2);
}
```

`PartialOrd` refines `PartialEq` in the sense that when comparing `PartialEq` types we can check if they are equal or not equal, but when comparing `PartialOrd` types we can check if they are equal or not equal, and if they are not equal we can check if they are unequal because the first item is less than or greater than the second item.

By default `Rhs = Self` because we almost always want to compare instances of a type to each other, and not to instances of different types. This also automatically guarantees our impl is symmetric and transitive.

```rust
use std::cmp::Ordering;
#[derive(PartialEq, PartialOrd)]
struct Point {
    x: i32,
    y: i32
}
// Rhs == Self == Point
impl PartialOrd for Point {
    // impl automatically symmetric & transitive
    fn partial_cmp(&self, other: &Point) -> Option<Ordering> {
        Some(match self.x.cmp(&other.x) {
            Ordering::Equal => self.y.cmp(&other.y),
            ordering => ordering,
        })
    }
}
```

If all the members of a type impl `PartialOrd` then it can be derived:

```rust
#[derive(PartialEq, PartialOrd)]
struct Point {
    x: i32,
    y: i32,
}
#[derive(PartialEq, PartialOrd)]
enum Stoplight {
    Red,
    Yellow,
    Green,
}
```

The `PartialOrd` derive macro orders types based on the lexicographical order of their members:

```rust
// generates PartialOrd impl which orders
// Points based on x member first and
// y member second because that's the order
// they appear in the source code
#[derive(PartialOrd, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
// generates DIFFERENT PartialOrd impl
// which orders Points based on y member
// first and x member second
#[derive(PartialOrd, PartialEq)]
struct Point {
    y: i32,
    x: i32,
}
```

`Ord` is a subtrait of `Eq` and `PartialOrd<Self>`:

```rust
trait Ord: Eq + PartialOrd<Self> {
    fn cmp(&self, other: &Self) -> Ordering;
    // provided default impls
    fn max(self, other: Self) -> Self;
    fn min(self, other: Self) -> Self;
    fn clamp(self, min: Self, max: Self) -> Self;
}
```

If we impl `Ord` for a type, on top of the asymmetry & transitivity properties required by `PartialOrd`, we're also guaranteeing that the asymmetry is total, i.e. exactly one of `a < b`, `a == b` or `a > b` is true for any given `a` and `b`. In this sense `Ord` refines `Eq` and `PartialOrd` because it represents a stricter version of comparisons. If a type impls `Ord` we can use that impl to trivially impl `PartialOrd`, `PartialEq`, and `Eq`:

```rust
use std::cmp::Ordering;
// of course we can use the derive macros here
#[derive(Ord, PartialOrd, Eq, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
// note: as with PartialOrd, the Ord derive macro
// orders a type based on the lexicographical order
// of its members
// but here's the impls if we wrote them out by hand
impl Ord for Point {
    fn cmp(&self, other: &Self) -> Ordering {
        match self.x.cmp(&other.x) {
            Ordering::Equal => self.y.cmp(&other.y),
            ordering => ordering,
        }
    }
}
impl PartialOrd for Point {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.cmp(other))
    }
}
impl PartialEq for Point {
    fn eq(&self, other: &Self) -> bool {
        self.cmp(other) == Ordering::Equal
    }
}
impl Eq for Point {}
```

Floats impl `PartialOrd` but not `Ord` because both `NaN < 0 == false` and `NaN >= 0 == false` are simultaneously true. Almost all other `PartialOrd` types are trivially `Ord`, unless of course if they contain floats.

Once a type impls `Ord` we can store it in `BTreeMap`s and `BTreeSet`s as well as easily sort it using the `sort()` method on slices and any types which deref to slices such as arrays, `Vec`s, and `VecDeque`s.

```rust
use std::collections::BTreeSet;
// now our type can be stored
// in BTreeSets and BTreeMaps!
#[derive(Ord, PartialOrd, PartialEq, Eq)]
struct Point {
    x: i32,
    y: i32,
}
fn example_btreeset() {
    let mut points = BTreeSet::new();
    points.insert(Point { x: 0, y: 0 }); // ✅
}
// we can also .sort() Ord types in collections!
fn example_sort<T: Ord>(mut sortable: Vec<T>) -> Vec<T> {
    sortable.sort();
    sortable
}
```

#### Index & IndexMut

Prerequisites

- [Self](#self)
- [Methods](#methods)
- [Associated Types](#associated-types)
- [Generic Parameters](#generic-parameters)
- [Generic Types vs Associated Types](#generic-types-vs-associated-types)
- [Subtraits & Supertraits](#subtraits--supertraits)
- [Sized](#sized)

```rust
trait Index<Idx: ?Sized> {
    type Output: ?Sized;
    fn index(&self, index: Idx) -> &Self::Output;
}
trait IndexMut<Idx>: Index<Idx> where Idx: ?Sized {
    fn index_mut(&mut self, index: Idx) -> &mut Self::Output;
}
```

We can index `[]` into `Index<T, Output = U>` types with `T` values and the index operation will return `&U` values. For syntax sugar, the compiler auto inserts a deref operator `*` in front of any value returned from an index operation:

```rust
fn main() {
    // Vec<i32> impls Index<usize, Output = i32> so
    // indexing Vec<i32> should produce &i32s and yet...
    let vec = vec![1, 2, 3, 4, 5];
    let num_ref: &i32 = vec[0]; // ❌ expected &i32 found i32

    // above line actually desugars to
    let num_ref: &i32 = *vec[0]; // ❌ expected &i32 found i32
    // both of these alternatives work
    let num: i32 = vec[0]; // ✅
    let num_ref = &vec[0]; // ✅
}
```

It's kinda confusing at first, because it seems like the `Index` trait does not follow its own method signature, but really it's just questionable syntax sugar.

Since `Idx` is a generic type the `Index` trait can be implemented many times for a given type, and in the case of `Vec<T>` not only can we index into it using `usize` but we can also index into it using `Range<usize>`s to get slices.

```rust
fn main() {
    let vec = vec![1, 2, 3, 4, 5];
    assert_eq!(&vec[..], &[1, 2, 3, 4, 5]); // ✅
    assert_eq!(&vec[1..], &[2, 3, 4, 5]); // ✅
    assert_eq!(&vec[..4], &[1, 2, 3, 4]); // ✅
    assert_eq!(&vec[1..4], &[2, 3, 4]); // ✅
}
```

To show off how we might impl `Index` ourselves here's a fun example which shows how we can use a newtype and the `Index` trait to impl wrapping indexes and negative indexes on a `Vec`:

```rust
use std::ops::Index;
struct WrappingIndex<T>(Vec<T>);
impl<T> Index<usize> for WrappingIndex<T> {
    type Output = T;
    fn index(&self, index: usize) -> &T {
        &self.0[index % self.0.len()]
    }
}
impl<T> Index<i128> for WrappingIndex<T> {
    type Output = T;
    fn index(&self, index: i128) -> &T {
        let self_len = self.0.len() as i128;
        let idx = (((index % self_len) + self_len) % self_len) as usize;
        &self.0[idx]
    }
}
#[test] // ✅
fn indexes() {
    let wrapping_vec = WrappingIndex(vec![1, 2, 3]);
    assert_eq!(1, wrapping_vec[0_usize]);
    assert_eq!(2, wrapping_vec[1_usize]);
    assert_eq!(3, wrapping_vec[2_usize]);
}
#[test] // ✅
fn wrapping_indexes() {
    let wrapping_vec = WrappingIndex(vec![1, 2, 3]);
    assert_eq!(1, wrapping_vec[3_usize]);
    assert_eq!(2, wrapping_vec[4_usize]);
    assert_eq!(3, wrapping_vec[5_usize]);
}
#[test] // ✅
fn neg_indexes() {
    let wrapping_vec = WrappingIndex(vec![1, 2, 3]);
    assert_eq!(1, wrapping_vec[-3_i128]);
    assert_eq!(2, wrapping_vec[-2_i128]);
    assert_eq!(3, wrapping_vec[-1_i128]);
}
#[test] // ✅
fn wrapping_neg_indexes() {
    let wrapping_vec = WrappingIndex(vec![1, 2, 3]);
    assert_eq!(1, wrapping_vec[-6_i128]);
    assert_eq!(2, wrapping_vec[-5_i128]);
    assert_eq!(3, wrapping_vec[-4_i128]);
}
```

There's no requirement that the `Idx` type has to be a number type or a `Range`, it could be an enum! Here's an example using basketball positions to index into a basketball team to retrieve players on the team:

```rust
use std::ops::Index;
enum BasketballPosition {
    PointGuard,
    ShootingGuard,
    Center,
    PowerForward,
    SmallForward,
}
struct BasketballPlayer {
    name: &'static str,
    position: BasketballPosition,
}
struct BasketballTeam {
    point_guard: BasketballPlayer,
    shooting_guard: BasketballPlayer,
    center: BasketballPlayer,
    power_forward: BasketballPlayer,
    small_forward: BasketballPlayer,
}
impl Index<BasketballPosition> for BasketballTeam {
    type Output = BasketballPlayer;
    fn index(&self, position: BasketballPosition) -> &BasketballPlayer {
        match position {
            BasketballPosition::PointGuard => &self.point_guard,
            BasketballPosition::ShootingGuard => &self.shooting_guard,
            BasketballPosition::Center => &self.center,
            BasketballPosition::PowerForward => &self.power_forward,
            BasketballPosition::SmallForward => &self.small_forward,
        }
    }
}
```

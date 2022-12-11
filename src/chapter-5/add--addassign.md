#### Add & AddAssign

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Associated Types](../chapter-1/associated-types.md)
- [Generic Parameters](../chapter-1/generic-parameters.md)
- [Generic Types vs Associated Types](../chapter-1/generic-types-vs-associated-types.md)
- [Derive Macros](../chapter-1/derive-macros.md)

```rust
trait Add<Rhs = Self> {
    type Output;
    fn add(self, rhs: Rhs) -> Self::Output;
}
```

`Add<Rhs, Output = T>` types can be added to `Rhs` types and will produce `T` as output.

Example `Add<Point, Output = Point>` impl for `Point`:

```rust
#[derive(Clone, Copy)]
struct Point {
    x: i32,
    y: i32,
}
impl Add for Point {
    type Output = Point;
    fn add(self, rhs: Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let p3 = p1 + p2;
    assert_eq!(p3.x, p1.x + p2.x); // ✅
    assert_eq!(p3.y, p1.y + p2.y); // ✅
}
```

But what if we only had references to `Point`s? Can we still add them then? Let's try:

```rust
fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let p3 = &p1 + &p2; // ❌
}
```

Unfortunately not. The compiler throws:

```none
error[E0369]: cannot add `&Point` to `&Point`
  --> src/main.rs:50:25
   |
50 |     let p3: Point = &p1 + &p2;
   |                     --- ^ --- &Point
   |                     |
   |                     &Point
   |
   = note: an implementation of `std::ops::Add` might be missing for `&Point`
```

Within Rust's type system, for some type `T`, the types `T`, `&T`, and `&mut T` are all treated as unique distinct types which means we have to provide trait impls for each of them separately. Let's define an `Add` impl for `&Point`:

```rust
impl Add for &Point {
    type Output = Point;
    fn add(self, rhs: &Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let p3 = &p1 + &p2; // ✅
    assert_eq!(p3.x, p1.x + p2.x); // ✅
    assert_eq!(p3.y, p1.y + p2.y); // ✅
}
```

However, something still doesn't feel quite right. We have two separate impls of `Add` for `Point` and `&Point` and they _happen_ to do the same thing currently but there's no guarantee that they will in the future! For example, let's say we decide that when we add two `Point`s together we want to create a `Line` containing those two `Point`s instead of creating a new `Point`, we'd update our `Add` impl like this:

```rust
use std::ops::Add;
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}
#[derive(Copy, Clone)]
struct Line {
    start: Point,
    end: Point,
}
// we updated this impl
impl Add for Point {
    type Output = Line;
    fn add(self, rhs: Point) -> Line {
        Line {
            start: self,
            end: rhs,
        }
    }
}
// but forgot to update this impl, uh oh!
impl Add for &Point {
    type Output = Point;
    fn add(self, rhs: &Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let line: Line = p1 + p2; // ✅
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let line: Line = &p1 + &p2; // ❌ expected Line, found Point
}
```

Our current impl of `Add` for `&Point` creates an unnecessary maintenance burden, we want the impl to match `Point`'s impl without having to manually update it every time we change `Point`'s impl. We'd like to keep our code as DRY (Don't Repeat Yourself) as possible. Luckily this is achievable:

```rust
// updated, DRY impl
impl Add for &Point {
    type Output = <Point as Add>::Output;
    fn add(self, rhs: &Point) -> Self::Output {
        Point::add(*self, *rhs)
    }
}
fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let line: Line = p1 + p2; // ✅
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let line: Line = &p1 + &p2; // ✅
}
```

`AddAssign<Rhs>` types allow us to add + assign `Rhs` types to them. The trait declaration:

```rust
trait AddAssign<Rhs = Self> {
    fn add_assign(&mut self, rhs: Rhs);
}
```

Example impls for `Point` and `&Point`:

```rust
use std::ops::AddAssign;
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32
}
impl AddAssign for Point {
    fn add_assign(&mut self, rhs: Point) {
        self.x += rhs.x;
        self.y += rhs.y;
    }
}
impl AddAssign<&Point> for Point {
    fn add_assign(&mut self, rhs: &Point) {
        Point::add_assign(self, *rhs);
    }
}
fn main() {
    let mut p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    p1 += &p2;
    p1 += p2;
    assert!(p1.x == 7 && p1.y == 10);
}
```

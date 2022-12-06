#### Generic Types vs Associated Types

Both generic types and associated types defer the decision to the implementer on which concrete types should be used in the trait's functions and methods, so this section seeks to explain when to use one over the other.

The general rule-of-thumb is:

- Use associated types when there should only be a single impl of the trait per type.
- Use generic types when there can be many possible impls of the trait per type.

Let's say we want to define a trait called `Add` which allows us to add values together. Here's an initial design and impl that only uses associated types:

```rust
trait Add {
    type Rhs;
    type Output;
    fn add(self, rhs: Self::Rhs) -> Self::Output;
}
struct Point {
    x: i32,
    y: i32,
}
impl Add for Point {
    type Rhs = Point;
    type Output = Point;
    fn add(self, rhs: Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 1 };
    let p2 = Point { x: 2, y: 2 };
    let p3 = p1.add(p2);
    assert_eq!(p3.x, 3);
    assert_eq!(p3.y, 3);
}
```

Let's say we wanted to add the ability to add `i32`s to `Point`s where the `i32` would be added to both the `x` and `y` members:

```rust
trait Add {
    type Rhs;
    type Output;
    fn add(self, rhs: Self::Rhs) -> Self::Output;
}
struct Point {
    x: i32,
    y: i32,
}
impl Add for Point {
    type Rhs = Point;
    type Output = Point;
    fn add(self, rhs: Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
impl Add for Point { // ❌
    type Rhs = i32;
    type Output = Point;
    fn add(self, rhs: i32) -> Point {
        Point {
            x: self.x + rhs,
            y: self.y + rhs,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 1 };
    let p2 = Point { x: 2, y: 2 };
    let p3 = p1.add(p2);
    assert_eq!(p3.x, 3);
    assert_eq!(p3.y, 3);

    let p1 = Point { x: 1, y: 1 };
    let int2 = 2;
    let p3 = p1.add(int2); // ❌
    assert_eq!(p3.x, 3);
    assert_eq!(p3.y, 3);
}
```

Throws:

```none
error[E0119]: conflicting implementations of trait `Add` for type `Point`:
  --> src/main.rs:23:1
   |
12 | impl Add for Point {
   | ------------------ first implementation here
...
23 | impl Add for Point {
   | ^^^^^^^^^^^^^^^^^^ conflicting implementation for `Point`
```

Since the `Add` trait is not parameterized by any generic types we can only impl it once per type, which means we can only pick the types for both `Rhs` and `Output` once! To allow adding both `Points`s and `i32`s to `Point` we have to refactor `Rhs` from an associated type to a generic type, which would allow us to impl the trait multiple times for `Point` with different type arguments for `Rhs`:

```rust
trait Add<Rhs> {
    type Output;
    fn add(self, rhs: Rhs) -> Self::Output;
}
struct Point {
    x: i32,
    y: i32,
}
impl Add<Point> for Point {
    type Output = Self;
    fn add(self, rhs: Point) -> Self::Output {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
impl Add<i32> for Point { // ✅
    type Output = Self;
    fn add(self, rhs: i32) -> Self::Output {
        Point {
            x: self.x + rhs,
            y: self.y + rhs,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 1 };
    let p2 = Point { x: 2, y: 2 };
    let p3 = p1.add(p2);
    assert_eq!(p3.x, 3);
    assert_eq!(p3.y, 3);

    let p1 = Point { x: 1, y: 1 };
    let int2 = 2;
    let p3 = p1.add(int2); // ✅
    assert_eq!(p3.x, 3);
    assert_eq!(p3.y, 3);
}
```

Let's say we add a new type called `Line` which contains two `Point`s, and now there are contexts within our program where adding two `Point`s should produce a `Line` instead of a `Point`. This is not possible given the current design of the `Add` trait where `Output` is an associated type but we can satisfy these new requirements by refactoring `Output` from an associated type into a generic type:

```rust
trait Add<Rhs, Output> {
    fn add(self, rhs: Rhs) -> Output;
}
struct Point {
    x: i32,
    y: i32,
}
impl Add<Point, Point> for Point {
    fn add(self, rhs: Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}
impl Add<i32, Point> for Point {
    fn add(self, rhs: i32) -> Point {
        Point {
            x: self.x + rhs,
            y: self.y + rhs,
        }
    }
}
struct Line {
    start: Point,
    end: Point,
}
impl Add<Point, Line> for Point { // ✅
    fn add(self, rhs: Point) -> Line {
        Line {
            start: self,
            end: rhs,
        }
    }
}
fn main() {
    let p1 = Point { x: 1, y: 1 };
    let p2 = Point { x: 2, y: 2 };
    let p3: Point = p1.add(p2);
    assert!(p3.x == 3 && p3.y == 3);
    let p1 = Point { x: 1, y: 1 };
    let int2 = 2;
    let p3 = p1.add(int2);
    assert!(p3.x == 3 && p3.y == 3);
    let p1 = Point { x: 1, y: 1 };
    let p2 = Point { x: 2, y: 2 };
    let l: Line = p1.add(p2); // ✅
    assert!(l.start.x == 1 && l.start.y == 1 && l.end.x == 2 && l.end.y == 2)
}
```

So which `Add` trait above is the best? It really depends on the requirements of your program! They're all good in the right situations.

### From & Into

Prerequisites

- [Self](#self)
- [Functions](#functions)
- [Methods](#methods)
- [Generic Parameters](#generic-parameters)
- [Generic Blanket Impls](#generic-blanket-impls)

```rust
trait From<T> {
    fn from(T) -> Self;
}
```

`From<T>` types allow us to convert `T` into `Self`.

```rust
trait Into<T> {
    fn into(self) -> T;
}
```

`Into<T>` types allow us to convert `Self` into `T`.

These traits are two different sides of the same coin. We can only impl `From<T>` for our types because the `Into<T>` impl is automatically provided by this generic blanket impl:

```rust
impl<T, U> Into<U> for T
where
    U: From<T>,
{
    fn into(self) -> U {
        U::from(self)
    }
}
```

The reason both traits exist is because it allows us to write trait bounds on generic types slightly differently:

```rust
fn function<T>(t: T)
where
    // these bounds are equivalent
    T: From<i32>,
    i32: Into<T>
{
    // these examples are equivalent
    let example: T = T::from(0);
    let example: T = 0.into();
}
```

There are no hard rules about when to use one or the other, so go with whatever makes the most sense for each situation. Now let's look at some example impls on `Point`:

```rust
struct Point {
    x: i32,
    y: i32,
}
impl From<(i32, i32)> for Point {
    fn from((x, y): (i32, i32)) -> Self {
        Point { x, y }
    }
}
impl From<[i32; 2]> for Point {
    fn from([x, y]: [i32; 2]) -> Self {
        Point { x, y }
    }
}
fn example() {
    // using From
    let origin = Point::from((0, 0));
    let origin = Point::from([0, 0]);
    // using Into
    let origin: Point = (0, 0).into();
    let origin: Point = [0, 0].into();
}
```

The impl is not symmetric, so if we'd like to convert `Point`s into tuples and arrays we have to explicitly add those as well:

```rust
struct Point {
    x: i32,
    y: i32,
}
impl From<(i32, i32)> for Point {
    fn from((x, y): (i32, i32)) -> Self {
        Point { x, y }
    }
}
impl From<Point> for (i32, i32) {
    fn from(Point { x, y }: Point) -> Self {
        (x, y)
    }
}
impl From<[i32; 2]> for Point {
    fn from([x, y]: [i32; 2]) -> Self {
        Point { x, y }
    }
}
impl From<Point> for [i32; 2] {
    fn from(Point { x, y }: Point) -> Self {
        [x, y]
    }
}
fn example() {
    // from (i32, i32) into Point
    let point = Point::from((0, 0));
    let point: Point = (0, 0).into();
    // from Point into (i32, i32)
    let tuple = <(i32, i32)>::from(point);
    let tuple: (i32, i32) = point.into();
    // from [i32; 2] into Point
    let point = Point::from([0, 0]);
    let point: Point = [0, 0].into();
    // from Point into [i32; 2]
    let array = <[i32; 2]>::from(point);
    let array: [i32; 2] = point.into();
}
```

A popular use of `From<T>` is to trim down boilerplate code. Let's say we add a `Triangle` type to our program which contains three `Point`s, here's some of the many ways we can construct it:

```rust
struct Point {
    x: i32,
    y: i32,
}
impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x, y }
    }
}
impl From<(i32, i32)> for Point {
    fn from((x, y): (i32, i32)) -> Point {
        Point { x, y }
    }
}
struct Triangle {
    p1: Point,
    p2: Point,
    p3: Point,
}
impl Triangle {
    fn new(p1: Point, p2: Point, p3: Point) -> Triangle {
        Triangle { p1, p2, p3 }
    }
}
impl<P> From<[P; 3]> for Triangle
where
    P: Into<Point>
{
    fn from([p1, p2, p3]: [P; 3]) -> Triangle {
        Triangle {
            p1: p1.into(),
            p2: p2.into(),
            p3: p3.into(),
        }
    }
}
fn example() {
    // manual construction
    let triangle = Triangle {
        p1: Point {
            x: 0,
            y: 0,
        },
        p2: Point {
            x: 1,
            y: 1,
        },
        p3: Point {
            x: 2,
            y: 2,
        },
    };
    // using Point::new
    let triangle = Triangle {
        p1: Point::new(0, 0),
        p2: Point::new(1, 1),
        p3: Point::new(2, 2),
    };
    // using From<(i32, i32)> for Point
    let triangle = Triangle {
        p1: (0, 0).into(),
        p2: (1, 1).into(),
        p3: (2, 2).into(),
    };
    // using Triangle::new + From<(i32, i32)> for Point
    let triangle = Triangle::new(
        (0, 0).into(),
        (1, 1).into(),
        (2, 2).into(),
    );
    // using From<[Into<Point>; 3]> for Triangle
    let triangle: Triangle = [
        (0, 0),
        (1, 1),
        (2, 2),
    ].into();
}
```

There are no rules for when, how, or why we should impl `From<T>` for our types so it's up to us to use our best judgement for every situation.

One popular use of `Into<T>` is to make functions which need owned values generic over whether they take owned or borrowed values:

```rust
struct Person {
    name: String,
}
impl Person {
    // accepts:
    // - String
    fn new1(name: String) -> Person {
        Person { name }
    }
    // accepts:
    // - String
    // - &String
    // - &str
    // - Box<str>
    // - Cow<'_, str>
    // - char
    // since all of the above types can be converted into String
    fn new2<N: Into<String>>(name: N) -> Person {
        Person { name: name.into() }
    }
}
```

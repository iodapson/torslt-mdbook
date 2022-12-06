### Trait Objects

Generics give us compile-time polymorphism where trait objects give us run-time polymorphism. We can use trait objects to allow functions to dynamically return different types at run-time:

```rust
fn example(condition: bool, vec: Vec<i32>) -> Box<dyn Iterator<Item = i32>> {
    let iter = vec.into_iter();
    if condition {
        // Has type:
        // Box<Map<IntoIter<i32>, Fn(i32) -> i32>>
        // But is cast to:
        // Box<dyn Iterator<Item = i32>>
        Box::new(iter.map(|n| n * 2))
    } else {
        // Has type:
        // Box<Filter<IntoIter<i32>, Fn(&i32) -> bool>>
        // But is cast to:
        // Box<dyn Iterator<Item = i32>>
        Box::new(iter.filter(|&n| n >= 2))
    }
}
```

Trait objects also allow us to store heterogeneous types in collections:

```rust
use std::f64::consts::PI;
struct Circle {
    radius: f64,
}
struct Square {
    side: f64
}
trait Shape {
    fn area(&self) -> f64;
}
impl Shape for Circle {
    fn area(&self) -> f64 {
        PI * self.radius * self.radius
    }
}
impl Shape for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }
}
fn get_total_area(shapes: Vec<Box<dyn Shape>>) -> f64 {
    shapes.into_iter().map(|s| s.area()).sum()
}
fn example() {
    let shapes: Vec<Box<dyn Shape>> = vec![
        Box::new(Circle { radius: 1.0 }), // Box<Circle> cast to Box<dyn Shape>
        Box::new(Square { side: 1.0 }), // Box<Square> cast to Box<dyn Shape>
    ];
    assert_eq!(PI + 1.0, get_total_area(shapes)); // ✅
}
```

Trait objects are unsized so they must always be behind a pointer. We can tell the difference between a concrete type and a trait object at the type level based on the presence of the `dyn` keyword within the type:

```rust
struct Struct;
trait Trait {}
// regular struct
&Struct
Box<Struct>
Rc<Struct>
Arc<Struct>
// trait objects
&dyn Trait
Box<dyn Trait>
Rc<dyn Trait>
Arc<dyn Trait>
```

Not all traits can be converted into trait objects. A trait is object-safe if it meets these requirements:

- trait doesn't require `Self: Sized`
- all of the trait's methods are object-safe

A trait method is object-safe if it meets these requirements:

- method requires `Self: Sized` or
- method only uses a `Self` type in receiver position

Understanding why the requirements are what they are is not relevant to the rest of this article, but if you're still curious it's covered in [Sizedness in Rust](https://github.com/pretzelhammer/rust-blog/blob/master/posts/sizedness-in-rust.md).

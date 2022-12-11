### Any

Prerequisites

- [Self](../chapter-1/self.md)
- [Generic Blanket Impls](../chapter-1/generic-blanket-impls.md)
- [Subtraits & Supertraits](../chapter-1/subtraits--supertraits.md)
- [Trait Objects](../chapter-1/trait-objects.md)

```rust
trait Any: 'static {
    fn type_id(&self) -> TypeId;
}
```

Rust's style of polymorphism is parametric, but if we're looking to use a more ad-hoc style of polymorphism similar to dynamically-typed languages then we can emulate that using the `Any` trait. We don't have to manually impl this trait for our types because that's already covered by this generic blanket impl:

```rust
impl<T: 'static + ?Sized> Any for T {
    fn type_id(&self) -> TypeId {
        TypeId::of::<T>()
    }
}
```

The way we get a `T` out of a `dyn Any` is by using the `downcast_ref::<T>()` and `downcast_mut::<T>()` methods:

```rust
use std::any::Any;
#[derive(Default)]
struct Point {
    x: i32,
    y: i32,
}
impl Point {
    fn inc(&mut self) {
        self.x += 1;
        self.y += 1;
    }
}
fn map_any(mut any: Box<dyn Any>) -> Box<dyn Any> {
    if let Some(num) = any.downcast_mut::<i32>() {
        *num += 1;
    } else if let Some(string) = any.downcast_mut::<String>() {
        *string += "!";
    } else if let Some(point) = any.downcast_mut::<Point>() {
        point.inc();
    }
    any
}
fn main() {
    let mut vec: Vec<Box<dyn Any>> = vec![
        Box::new(0),
        Box::new(String::from("a")),
        Box::new(Point::default()),
    ];
    // vec = [0, "a", Point { x: 0, y: 0 }]
    vec = vec.into_iter().map(map_any).collect();
    // vec = [1, "a!", Point { x: 1, y: 1 }]
}
```

This trait rarely _needs_ to be used because on top of parametric polymorphism being superior to ad-hoc polymorphism in most scenarios the latter can also be emulated using enums which are more type-safe and require less indirection. For example, we could have written the above example like this:

```rust
#[derive(Default)]
struct Point {
    x: i32,
    y: i32,
}
impl Point {
    fn inc(&mut self) {
        self.x += 1;
        self.y += 1;
    }
}
enum Stuff {
    Integer(i32),
    String(String),
    Point(Point),
}
fn map_stuff(mut stuff: Stuff) -> Stuff {
    match &mut stuff {
        Stuff::Integer(num) => *num += 1,
        Stuff::String(string) => *string += "!",
        Stuff::Point(point) => point.inc(),
    }
    stuff
}
fn main() {
    let mut vec = vec![
        Stuff::Integer(0),
        Stuff::String(String::from("a")),
        Stuff::Point(Point::default()),
    ];
    // vec = [0, "a", Point { x: 0, y: 0 }]
    vec = vec.into_iter().map(map_stuff).collect();
    // vec = [1, "a!", Point { x: 1, y: 1 }]
}
```

Despite `Any` rarely being _needed_ it can still be convenient to use sometimes, as we'll later see in the **Error Handling** section.

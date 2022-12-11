### Default

Prerequisites

- [Self](../chapter-1/self.md)
- [Functions](../chapter-1/functions.md)
- [Derive Macros](../chapter-1/derive-macros.md)

```rust
trait Default {
    fn default() -> Self;
}
```

It's possible to construct default values of `Default` types.

```rust
struct Color {
    r: u8,
    g: u8,
    b: u8,
}
impl Default for Color {
    // default color is black
    fn default() -> Self {
        Color {
            r: 0,
            g: 0,
            b: 0,
        }
    }
}
```

This is useful for quick prototyping but also in any instance where we just need an instance of a type and aren't picky about what it is:

```rust
fn main() {
    // just give me some color!
    let color = Color::default();
}
```

This is also an option we may want to explicitly expose to the users of our functions:

```rust
struct Canvas;
enum Shape {
    Circle,
    Rectangle,
}
impl Canvas {
    // let user optionally pass a color
    fn paint(&mut self, shape: Shape, color: Option<Color>) {
        // if no color is passed use the default color
        let color = color.unwrap_or_default();
        // etc
    }
}
```

`Default` is also useful in generic contexts where we need to construct generic types:

```rust
fn guarantee_length<T: Default>(mut vec: Vec<T>, min_len: usize) -> Vec<T> {
    for _ in 0..min_len.saturating_sub(vec.len()) {
        vec.push(T::default());
    }
    vec
}
```

Another way we can take advantage of `Default` types is for partial initialization of structs using Rust's struct update syntax. We may have a `new` constructor for `Color` which takes every member as an argument:

```rust
impl Color {
    fn new(r: u8, g: u8, b: u8) -> Self {
        Color {
            r,
            g,
            b,
        }
    }
}
```

However we can also have convenience constructors that only accept a particular struct member each and fall back to the default values for the other struct members:

```rust
impl Color {
    fn red(r: u8) -> Self {
        Color {
            r,
            ..Color::default()
        }
    }
    fn green(g: u8) -> Self {
        Color {
            g,
            ..Color::default()
        }
    }
    fn blue(b: u8) -> Self {
        Color {
            b,
            ..Color::default()
        }
    }
}
```

There's also a `Default` derive macro for so we can write `Color` like this:

```rust
// default color is still black
// because u8::default() == 0
#[derive(Default)]
struct Color {
    r: u8,
    g: u8,
    b: u8
}
```

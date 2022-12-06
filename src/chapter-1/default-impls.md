### Default Impls

Traits can provide default impls for their functions and methods.

```rust
trait Trait {
    fn method(&self) {
        println!("default impl");
    }
}
struct SomeType;
struct OtherType;
// use default impl for Trait::method
impl Trait for SomeType {}
impl Trait for OtherType {
    // use our own impl for Trait::method
    fn method(&self) {
        println!("OtherType impl");
    }
}
fn main() {
    SomeType.method(); // prints "default impl"
    OtherType.method(); // prints "OtherType impl"
}
```

This is especially handy if some of the trait methods can be implemented solely using other trait methods.

```rust
trait Greet {
    fn greet(&self, name: &str) -> String;
    fn greet_loudly(&self, name: &str) -> String {
        self.greet(name) + "!"
    }
}
struct Hello;
struct Hola;
impl Greet for Hello {
    fn greet(&self, name: &str) -> String {
        format!("Hello {}", name)
    }
    // use default impl for greet_loudly
}
impl Greet for Hola {
    fn greet(&self, name: &str) -> String {
        format!("Hola {}", name)
    }
    // override default impl
    fn greet_loudly(&self, name: &str) -> String {
        let mut greeting = self.greet(name);
        greeting.insert_str(0, "¡");
        greeting + "!"
    }
}
fn main() {
    println!("{}", Hello.greet("John")); // prints "Hello John"
    println!("{}", Hello.greet_loudly("John")); // prints "Hello John!"
    println!("{}", Hola.greet("John")); // prints "Hola John"
    println!("{}", Hola.greet_loudly("John")); // prints "¡Hola John!"
}
```

Many traits in the standard library provide default impls for many of their methods.

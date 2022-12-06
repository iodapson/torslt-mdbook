#### Generic Parameters

_"Generic parameters"_ broadly refers to generic type parameters, generic lifetime parameters, and generic const parameters. Since all of those are a mouthful to say people commonly abbreviate them to _"generic types"_, _"lifetimes"_, and _"generic consts"_. Since generic consts are not used in any of the standard library traits we'll be covering they're outside the scope of this article.

We can generalize a trait declaration using parameters:

```rust
// trait declaration generalized with lifetime & type parameters
trait Trait<'a, T> {
    // signature uses generic type
    fn func1(arg: T);

    // signature uses lifetime
    fn func2(arg: &'a i32);

    // signature uses generic type & lifetime
    fn func3(arg: &'a T);
}
struct SomeType;
impl<'a> Trait<'a, i8> for SomeType {
    fn func1(arg: i8) {}
    fn func2(arg: &'a i32) {}
    fn func3(arg: &'a i8) {}
}
impl<'b> Trait<'b, u8> for SomeType {
    fn func1(arg: u8) {}
    fn func2(arg: &'b i32) {}
    fn func3(arg: &'b u8) {}
}
```

It's possible to provide default values for generic types. The most commonly used default value is `Self` but any type works:

```rust
// make T = Self by default
trait Trait<T = Self> {
    fn func(t: T) {}
}
// any type can be used as the default
trait Trait2<T = i32> {
    fn func2(t: T) {}
}
struct SomeType;
// omitting the generic type will
// cause the impl to use the default
// value, which is Self here
impl Trait for SomeType {
    fn func(t: SomeType) {}
}
// default value here is i32
impl Trait2 for SomeType {
    fn func2(t: i32) {}
}
// the default is overridable as we'd expect
impl Trait<String> for SomeType {
    fn func(t: String) {}
}
// overridable here too
impl Trait2<String> for SomeType {
    fn func2(t: String) {}
}
```

Aside from parameterizing the trait it's also possible to parameterize individual functions and methods:

```rust
trait Trait {
    fn func<'a, T>(t: &'a T);
}
```

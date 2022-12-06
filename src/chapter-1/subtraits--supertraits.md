### Subtraits & Supertraits

The "sub" in "subtrait" refers to subset and the "super" in "supertrait" refers to superset. If we have this trait declaration:

```rust
trait Subtrait: Supertrait {}
```

All of the types which impl `Subtrait` are a subset of all the types which impl `Supertrait`, or to put it in opposite but equivalent terms: all the types which impl `Supertrait` are a superset of all the types which impl `Subtrait`.

Also, the above is just syntax sugar for:

```rust
trait Subtrait where Self: Supertrait {}
```

It's a subtle yet important distinction to understand that the bound is on `Self`, i.e. the type impling `Subtrait`, and not on `Subtrait` itself. The latter would not make any sense, since trait bounds can only be applied to concrete types which can impl traits. Traits cannot impl other traits:

```rust
trait Supertrait {
    fn method(&self) {
        println!("in supertrait");
    }
}
trait Subtrait: Supertrait {
    // this looks like it might impl or
    // override Supertrait::method but it
    // does not
    fn method(&self) {
        println!("in subtrait")
    }
}
struct SomeType;
// adds Supertrait::method to SomeType
impl Supertrait for SomeType {}
// adds Subtrait::method to SomeType
impl Subtrait for SomeType {}
// both methods exist on SomeType simultaneously
// neither overriding or shadowing the other
fn main() {
    SomeType.method(); // ❌ ambiguous method call
    // must disambiguate using fully-qualified syntax
    <SomeType as Supertrait>::method(&SomeType); // ✅ prints "in supertrait"
    <SomeType as Subtrait>::method(&SomeType); // ✅ prints "in subtrait"
}
```

Furthermore, there are no rules for how a type must impl both a subtrait and a supertrait. It can use the methods from either in the impl of the other.

```rust
trait Supertrait {
    fn super_method(&mut self);
}
trait Subtrait: Supertrait {
    fn sub_method(&mut self);
}
struct CallSuperFromSub;
impl Supertrait for CallSuperFromSub {
    fn super_method(&mut self) {
        println!("in super");
    }
}
impl Subtrait for CallSuperFromSub {
    fn sub_method(&mut self) {
        println!("in sub");
        self.super_method();
    }
}
struct CallSubFromSuper;
impl Supertrait for CallSubFromSuper {
    fn super_method(&mut self) {
        println!("in super");
        self.sub_method();
    }
}
impl Subtrait for CallSubFromSuper {
    fn sub_method(&mut self) {
        println!("in sub");
    }
}
struct CallEachOther(bool);
impl Supertrait for CallEachOther {
    fn super_method(&mut self) {
        println!("in super");
        if self.0 {
            self.0 = false;
            self.sub_method();
        }
    }
}
impl Subtrait for CallEachOther {
    fn sub_method(&mut self) {
        println!("in sub");
        if self.0 {
            self.0 = false;
            self.super_method();
        }
    }
}
fn main() {
    CallSuperFromSub.super_method(); // prints "in super"
    CallSuperFromSub.sub_method(); // prints "in sub", "in super"

    CallSubFromSuper.super_method(); // prints "in super", "in sub"
    CallSubFromSuper.sub_method(); // prints "in sub"

    CallEachOther(true).super_method(); // prints "in super", "in sub"
    CallEachOther(true).sub_method(); // prints "in sub", "in super"
}
```

Hopefully the examples above show that the relationship between subtraits and supertraits can be complex. Before introducing a mental model that neatly encapsulates all of that complexity let's quickly review and establish the mental model we use for understanding trait bounds on generic types:

```rust
fn function<T: Clone>(t: T) {
    // impl
}
```

Without knowing anything about the impl of this function we could reasonably guess that `t.clone()` gets called at some point because when a generic type is bounded by a trait that strongly implies it has a dependency on the trait. The mental model for understanding the relationship between generic types and their trait bounds is a simple and intuitive one: generic types _depend on_ their trait bounds.

Now let's look the trait declaration for `Copy`:

```rust
trait Copy: Clone {}
```

The syntax above looks very similar to the syntax for applying a trait bound on a generic type and yet `Copy` doesn't depend on `Clone` at all. The mental model we developed earlier doesn't help us here. In my opinion, the most simple and elegant mental model for understanding the relationship between subtraits and supertraits is: subtraits _refine_ their supertraits.

"Refinement" is intentionally kept somewhat vague because it can mean different things in different contexts:

- a subtrait might make its supertrait's methods' impls more specialized, faster, use less memory, e.g. `Copy: Clone`
- a subtrait might make additional guarantees about the supertrait's methods' impls, e.g. `Eq: PartialEq`, `Ord: PartialOrd`, `ExactSizeIterator: Iterator`
- a subtrait might make the supertrait's methods more flexible or easier to call, e.g. `FnMut: FnOnce`, `Fn: FnMut`
- a subtrait might extend a supertrait and add new methods, e.g. `DoubleEndedIterator: Iterator`, `ExactSizeIterator: Iterator`

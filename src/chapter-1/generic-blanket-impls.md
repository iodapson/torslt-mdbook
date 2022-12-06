### Generic Blanket Impls

A generic blanket impl is an impl on a generic type instead of a concrete type. To explain why and how we'd use one let's start by writing an `is_even` method for number types:

```rust
trait Even {
    fn is_even(self) -> bool;
}
impl Even for i8 {
    fn is_even(self) -> bool {
        self % 2_i8 == 0_i8
    }
}
impl Even for u8 {
    fn is_even(self) -> bool {
        self % 2_u8 == 0_u8
    }
}
impl Even for i16 {
    fn is_even(self) -> bool {
        self % 2_i16 == 0_i16
    }
}
// etc
#[test] // ✅
fn test_is_even() {
    assert!(2_i8.is_even());
    assert!(4_u8.is_even());
    assert!(6_i16.is_even());
    // etc
}
```

Obviously, this is very verbose. Also, all of our impls are almost identical. Furthermore, in the unlikely but still possible event that Rust decides to add more number types in the future we have to remember to come back to this code and update it with the new number types. We can solve all these problems using a generic blanket impl:

```rust
use std::fmt::Debug;
use std::convert::TryInto;
use std::ops::Rem;
trait Even {
    fn is_even(self) -> bool;
}
// generic blanket impl
impl<T> Even for T
where
    T: Rem<Output = T> + PartialEq<T> + Sized,
    u8: TryInto<T>,
    <u8 as TryInto<T>>::Error: Debug,
{
    fn is_even(self) -> bool {
        // these unwraps will never panic
        self % 2.try_into().unwrap() == 0.try_into().unwrap()
    }
}
#[test] // ✅
fn test_is_even() {
    assert!(2_i8.is_even());
    assert!(4_u8.is_even());
    assert!(6_i16.is_even());
    // etc
}
```

Unlike default impls, which provide _an_ impl, generic blanket impls provide _the_ impl, so they are not overridable.

```rust
use std::fmt::Debug;
use std::convert::TryInto;
use std::ops::Rem;
trait Even {
    fn is_even(self) -> bool;
}
impl<T> Even for T
where
    T: Rem<Output = T> + PartialEq<T> + Sized,
    u8: TryInto<T>,
    <u8 as TryInto<T>>::Error: Debug,
{
    fn is_even(self) -> bool {
        self % 2.try_into().unwrap() == 0.try_into().unwrap()
    }
}
impl Even for u8 { // ❌
    fn is_even(self) -> bool {
        self % 2_u8 == 0_u8
    }
}
```

Throws:

```none
error[E0119]: conflicting implementations of trait `Even` for type `u8`:
  --> src/lib.rs:22:1
   |
10 | / impl<T> Even for T
11 | | where
12 | |     T: Rem<Output = T> + PartialEq<T> + Sized,
13 | |     u8: TryInto<T>,
...  |
19 | |     }
20 | | }
   | |_- first implementation here
21 |
22 |   impl Even for u8 {
   |   ^^^^^^^^^^^^^^^^ conflicting implementation for `u8`
```

These impls overlap, hence they conflict, hence Rust rejects the code to ensure trait coherence. Trait coherence is the property that there exists at most one impl of a trait for any given type. The rules Rust uses to enforce trait coherence, the implications of those rules, and workarounds for the implications are outside the scope of this article.

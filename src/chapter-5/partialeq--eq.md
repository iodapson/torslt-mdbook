#### PartialEq & Eq

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Generic Parameters](../chapter-1/generic-parameters.md)
- [Default Impls](../chapter-1/default-impls.md)
- [Generic Blanket Impls](../chapter-1/generic-blanket-impls.md)
- [Marker Traits](../chapter-1/marker-traits.md)
- [Subtraits & Supertraits](../chapter-1/subtraits--supertraits.md)
- [Sized](../chapter-2/sized.md)

```rust
trait PartialEq<Rhs = Self>
where
    Rhs: ?Sized,
{
    fn eq(&self, other: &Rhs) -> bool;
    // provided default impls
    fn ne(&self, other: &Rhs) -> bool;
}
```

`PartialEq<Rhs>` types can be checked for equality to `Rhs` types using the `==` operator.

All `PartialEq<Rhs>` impls must ensure that equality is symmetric and transitive. That means for all `a`, `b`, and `c`:

- `a == b` implies `b == a` (symmetry)
- `a == b && b == c` implies `a == c` (transitivity)

By default `Rhs = Self` because we almost always want to compare instances of a type to each other, and not to instances of different types. This also automatically guarantees our impl is symmetric and transitive.

```rust
struct Point {
    x: i32,
    y: i32
}
// Rhs == Self == Point
impl PartialEq for Point {
    // impl automatically symmetric & transitive
    fn eq(&self, other: &Point) -> bool {
        self.x == other.x && self.y == other.y
    }
}
```

If all the members of a type impl `PartialEq` then it can be derived:

```rust
#[derive(PartialEq)]
struct Point {
    x: i32,
    y: i32
}
#[derive(PartialEq)]
enum Suit {
    Spade,
    Heart,
    Club,
    Diamond,
}
```

Once we impl `PartialEq` for our type we also get equality comparisons between references of our type for free thanks to these generic blanket impls:

```rust
// this impl only gives us: Point == Point
#[derive(PartialEq)]
struct Point {
    x: i32,
    y: i32
}
// all of the generic blanket impls below
// are provided by the standard library
// this impl gives us: &Point == &Point
impl<A, B> PartialEq<&'_ B> for &'_ A
where A: PartialEq<B> + ?Sized, B: ?Sized;
// this impl gives us: &mut Point == &Point
impl<A, B> PartialEq<&'_ B> for &'_ mut A
where A: PartialEq<B> + ?Sized, B: ?Sized;
// this impl gives us: &Point == &mut Point
impl<A, B> PartialEq<&'_ mut B> for &'_ A
where A: PartialEq<B> + ?Sized, B: ?Sized;
// this impl gives us: &mut Point == &mut Point
impl<A, B> PartialEq<&'_ mut B> for &'_ mut A
where A: PartialEq<B> + ?Sized, B: ?Sized;
```

Since this trait is generic we can define equality between different types. The standard library leverages this to allow checking equality between the many string-like types such as `String`, `&str`, `PathBuf`, `&Path`, `OsString`, `&OsStr`, and so on.
Generally, we should only impl equality between different types _if they contain the same kind of data_ and the only difference between the types is how they represent the data or how they allow interacting with the data.
Here's a cute but bad example of how someone might be tempted to impl `PartialEq` to check equality between different types that don't meet the above criteria:

```rust
#[derive(PartialEq)]
enum Suit {
    Spade,
    Club,
    Heart,
    Diamond,
}
#[derive(PartialEq)]
enum Rank {
    Ace,
    Two,
    Three,
    Four,
    Five,
    Six,
    Seven,
    Eight,
    Nine,
    Ten,
    Jack,
    Queen,
    King,
}
#[derive(PartialEq)]
struct Card {
    suit: Suit,
    rank: Rank,
}
// check equality of Card's suit
impl PartialEq<Suit> for Card {
    fn eq(&self, other: &Suit) -> bool {
        self.suit == *other
    }
}
// check equality of Card's rank
impl PartialEq<Rank> for Card {
    fn eq(&self, other: &Rank) -> bool {
        self.rank == *other
    }
}
fn main() {
    let AceOfSpades = Card {
        suit: Suit::Spade,
        rank: Rank::Ace,
    };
    assert!(AceOfSpades == Suit::Spade); // ✅
    assert!(AceOfSpades == Rank::Ace); // ✅
}
```

It works and kinda makes sense. A card which is an Ace of Spades is both an Ace and a Spade, and if we're writing a library to handle playing cards it's reasonable that we'd want to make it easy and convenient to individually check the suit and rank of a card. However, something's missing: symmetry! We can `Card == Suit` and `Card == Rank` but we cannot `Suit == Card` or `Rank == Card` so let's fix that:

```rust
// check equality of Card's suit
impl PartialEq<Suit> for Card {
    fn eq(&self, other: &Suit) -> bool {
        self.suit == *other
    }
}
// added for symmetry
impl PartialEq<Card> for Suit {
    fn eq(&self, other: &Card) -> bool {
        *self == other.suit
    }
}
// check equality of Card's rank
impl PartialEq<Rank> for Card {
    fn eq(&self, other: &Rank) -> bool {
        self.rank == *other
    }
}
// added for symmetry
impl PartialEq<Card> for Rank {
    fn eq(&self, other: &Card) -> bool {
        *self == other.rank
    }
}
```

We have symmetry! Great. Adding symmetry just broke transitivity! Oops. This is now possible:

```rust
fn main() {
    // Ace of Spades
    let a = Card {
        suit: Suit::Spade,
        rank: Rank::Ace,
    };
    let b = Suit::Spade;
    // King of Spades
    let c = Card {
        suit: Suit::Spade,
        rank: Rank::King,
    };
    assert!(a == b && b == c); // ✅
    assert!(a == c); // ❌
}
```

A good example of impling `PartialEq` to check equality between different types would be a program that works with distances and uses different types to represent different units of measurement.

```rust
#[derive(PartialEq)]
struct Foot(u32);
#[derive(PartialEq)]
struct Yard(u32);
#[derive(PartialEq)]
struct Mile(u32);
impl PartialEq<Mile> for Foot {
    fn eq(&self, other: &Mile) -> bool {
        self.0 == other.0 * 5280
    }
}
impl PartialEq<Foot> for Mile {
    fn eq(&self, other: &Foot) -> bool {
        self.0 * 5280 == other.0
    }
}
impl PartialEq<Mile> for Yard {
    fn eq(&self, other: &Mile) -> bool {
        self.0 == other.0 * 1760
    }
}
impl PartialEq<Yard> for Mile {
    fn eq(&self, other: &Yard) -> bool {
        self.0 * 1760 == other.0
    }
}
impl PartialEq<Foot> for Yard {
    fn eq(&self, other: &Foot) -> bool {
        self.0 * 3 == other.0
    }
}
impl PartialEq<Yard> for Foot {
    fn eq(&self, other: &Yard) -> bool {
        self.0 == other.0 * 3
    }
}
fn main() {
    let a = Foot(5280);
    let b = Yard(1760);
    let c = Mile(1);

    // symmetry
    assert!(a == b && b == a); // ✅
    assert!(b == c && c == b); // ✅
    assert!(a == c && c == a); // ✅
    // transitivity
    assert!(a == b && b == c && a == c); // ✅
    assert!(c == b && b == a && c == a); // ✅
}
```

`Eq` is a marker trait and a subtrait of `PartialEq<Self>`.

```rust
trait Eq: PartialEq<Self> {}
```

If we impl `Eq` for a type, on top of the symmetry & transitivity properties required by `PartialEq`, we're also guaranteeing reflexivity, i.e. `a == a` for all `a`. In this sense `Eq` refines `PartialEq` because it represents a stricter version of equality. If all members of a type impl `Eq` then the `Eq` impl can be derived for the type.

Floats are `PartialEq` but not `Eq` because `NaN != NaN`. Almost all other `PartialEq` types are trivially `Eq`, unless of course if they contain floats.

Once a type impls `PartialEq` and `Debug` we can use it in the `assert_eq!` macro. We can also compare collections of `PartialEq` types.

```rust
#[derive(PartialEq, Debug)]
struct Point {
    x: i32,
    y: i32,
}
fn example_assert(p1: Point, p2: Point) {
    assert_eq!(p1, p2);
}
fn example_compare_collections<T: PartialEq>(vec1: Vec<T>, vec2: Vec<T>) {
    // if T: PartialEq this now works!
    if vec1 == vec2 {
        // some code
    } else {
        // other code
    }
}
```

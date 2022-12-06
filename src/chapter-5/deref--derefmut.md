#### Deref & DerefMut

Prerequisites

- [Self](#self)
- [Methods](#methods)
- [Associated Types](#associated-types)
- [Subtraits & Supertraits](#subtraits--supertraits)
- [Sized](#sized)

```rust
trait Deref {
    type Target: ?Sized;
    fn deref(&self) -> &Self::Target;
}
trait DerefMut: Deref {
    fn deref_mut(&mut self) -> &mut Self::Target;
}
```

`Deref<Target = T>` types can be dereferenced to `T` types using the dereference operator `*`. This has obvious use-cases for smart pointer types like `Box` and `Rc`. However, we rarely see the dereference operator explicitly used in Rust code, and that's because of a Rust feature called _deref coercion_.

Rust automatically dereferences types when they're being passed as function arguments, returned from a function, or used as part of a method call. This is the reason why we can pass `&String` and `&Vec<T>` to functions expecting `&str` and `&[T]` because `String` impls `Deref<Target = str>` and `Vec<T>` impls `Deref<Target = [T]>`.

`Deref` and `DerefMut` should only be implemented for smart pointer types. The most common way people attempt to misuse and abuse these traits is to try to shoehorn some kind of OOP-style data inheritance into Rust. This does not work. Rust is not OOP. Let's examine a few different situations where, how, and why it does not work. Let's start with this example:

```rust
use std::ops::Deref;
struct Human {
    health_points: u32,
}
enum Weapon {
    Spear,
    Axe,
    Sword,
}
// a Soldier is just a Human with a Weapon
struct Soldier {
    human: Human,
    weapon: Weapon,
}
impl Deref for Soldier {
    type Target = Human;
    fn deref(&self) -> &Human {
        &self.human
    }
}
enum Mount {
    Horse,
    Donkey,
    Cow,
}
// a Knight is just a Soldier with a Mount
struct Knight {
    soldier: Soldier,
    mount: Mount,
}
impl Deref for Knight {
    type Target = Soldier;
    fn deref(&self) -> &Soldier {
        &self.soldier
    }
}
enum Spell {
    MagicMissile,
    FireBolt,
    ThornWhip,
}
// a Mage is just a Human who can cast Spells
struct Mage {
    human: Human,
    spells: Vec<Spell>,
}
impl Deref for Mage {
    type Target = Human;
    fn deref(&self) -> &Human {
        &self.human
    }
}
enum Staff {
    Wooden,
    Metallic,
    Plastic,
}
// a Wizard is just a Mage with a Staff
struct Wizard {
    mage: Mage,
    staff: Staff,
}
impl Deref for Wizard {
    type Target = Mage;
    fn deref(&self) -> &Mage {
        &self.mage
    }
}
fn borrows_human(human: &Human) {}
fn borrows_soldier(soldier: &Soldier) {}
fn borrows_knight(knight: &Knight) {}
fn borrows_mage(mage: &Mage) {}
fn borrows_wizard(wizard: &Wizard) {}
fn example(human: Human, soldier: Soldier, knight: Knight, mage: Mage, wizard: Wizard) {
    // all types can be used as Humans
    borrows_human(&human);
    borrows_human(&soldier);
    borrows_human(&knight);
    borrows_human(&mage);
    borrows_human(&wizard);
    // Knights can be used as Soldiers
    borrows_soldier(&soldier);
    borrows_soldier(&knight);
    // Wizards can be used as Mages
    borrows_mage(&mage);
    borrows_mage(&wizard);
    // Knights & Wizards passed as themselves
    borrows_knight(&knight);
    borrows_wizard(&wizard);
}
```

So at first glance the above looks pretty good! However it quickly breaks down to scrutiny. First of all, deref coercion only works on references, so it doesn't work when we actually want to pass ownership:

```rust
fn takes_human(human: Human) {}
fn example(human: Human, soldier: Soldier, knight: Knight, mage: Mage, wizard: Wizard) {
    // all types CANNOT be used as Humans
    takes_human(human);
    takes_human(soldier); // ❌
    takes_human(knight); // ❌
    takes_human(mage); // ❌
    takes_human(wizard); // ❌
}
```

Furthermore, deref coercion doesn't work in generic contexts. Let's say we impl some trait only on humans:

```rust
trait Rest {
    fn rest(&self);
}
impl Rest for Human {
    fn rest(&self) {}
}
fn take_rest<T: Rest>(rester: &T) {
    rester.rest()
}
fn example(human: Human, soldier: Soldier, knight: Knight, mage: Mage, wizard: Wizard) {
    // all types CANNOT be used as Rest types, only Human
    take_rest(&human);
    take_rest(&soldier); // ❌
    take_rest(&knight); // ❌
    take_rest(&mage); // ❌
    take_rest(&wizard); // ❌
}
```

Also, although deref coercion works in a lot of places it doesn't work everywhere. It doesn't work on operands, even though operators are just syntax sugar for method calls. Let's say, to be cute, we wanted `Mage`s to learn `Spell`s using the `+=` operator:

```rust
impl DerefMut for Wizard {
    fn deref_mut(&mut self) -> &mut Mage {
        &mut self.mage
    }
}
impl AddAssign<Spell> for Mage {
    fn add_assign(&mut self, spell: Spell) {
        self.spells.push(spell);
    }
}
fn example(mut mage: Mage, mut wizard: Wizard, spell: Spell) {
    mage += spell;
    wizard += spell; // ❌ wizard not coerced to mage here
    wizard.add_assign(spell); // oof, we have to call it like this 🤦
}
```

In languages with OOP-style data inheritance the value of `self` within a method is always equal to the type which called the method but in the case of Rust the value of `self` is always equal to the type which implemented the method:

```rust
struct Human {
    profession: &'static str,
    health_points: u32,
}
impl Human {
    // self will always be a Human here, even if we call it on a Soldier
    fn state_profession(&self) {
        println!("I'm a {}!", self.profession);
    }
}
struct Soldier {
    profession: &'static str,
    human: Human,
    weapon: Weapon,
}
fn example(soldier: &Soldier) {
    assert_eq!("servant", soldier.human.profession);
    assert_eq!("spearman", soldier.profession);
    soldier.human.state_profession(); // prints "I'm a servant!"
    soldier.state_profession(); // still prints "I'm a servant!" 🤦
}
```

The above gotcha is especially damning when impling `Deref` or `DerefMut` on a newtype. Let's say we want to create a `SortedVec` type which is just a `Vec` but it's always in sorted order. Here's how we might do that:

```rust
struct SortedVec<T: Ord>(Vec<T>);
impl<T: Ord> SortedVec<T> {
    fn new(mut vec: Vec<T>) -> Self {
        vec.sort();
        SortedVec(vec)
    }
    fn push(&mut self, t: T) {
        self.0.push(t);
        self.0.sort();
    }
}
```

Obviously we cannot impl `DerefMut<Target = Vec<T>>` here or anyone using `SortedVec` would be able to trivially break the sorted order. However, impling `Deref<Target = Vec<T>>` surely must be safe, right? Try to spot the bug in the program below:

```rust
use std::ops::Deref;
struct SortedVec<T: Ord>(Vec<T>);
impl<T: Ord> SortedVec<T> {
    fn new(mut vec: Vec<T>) -> Self {
        vec.sort();
        SortedVec(vec)
    }
    fn push(&mut self, t: T) {
        self.0.push(t);
        self.0.sort();
    }
}
impl<T: Ord> Deref for SortedVec<T> {
    type Target = Vec<T>;
    fn deref(&self) -> &Vec<T> {
        &self.0
    }
}
fn main() {
    let sorted = SortedVec::new(vec![2, 8, 6, 3]);
    sorted.push(1);
    let sortedClone = sorted.clone();
    sortedClone.push(4);
}
```

We never implemented `Clone` for `SortedVec` so when we call the `.clone()` method the compiler is using deref coercion to resolve that method call on `Vec` and so it returns a `Vec` and not a `SortedVec`!

```rust
fn main() {
    let sorted: SortedVec<i32> = SortedVec::new(vec![2, 8, 6, 3]);
    sorted.push(1); // still sorted
    // calling clone on SortedVec actually returns a Vec 🤦
    let sortedClone: Vec<i32> = sorted.clone();
    sortedClone.push(4); // sortedClone no longer sorted 💀
}
```

Anyway, none of the above limitations, constraints, or gotchas are faults of Rust because Rust was never designed to be an OO language or to support any OOP patterns in the first place.

The main takeaway from this section is do not try to be cute or clever with `Deref` and `DerefMut` impls. They're really only appropriate for smart pointer types, which can only be implemented within the standard library for now as smart pointer types currently require unstable features and compiler magic to work. If we want functionality and behavior similar to `Deref` and `DerefMut` then what we're actually probably looking for is `AsRef` and `AsMut` which we'll get to later.

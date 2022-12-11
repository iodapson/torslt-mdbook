### AsRef & AsMut

Prerequisites

- [Self](..chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Sized](../chapter-2/sized.md)
- [Generic Parameters](../chapter-1/generic-parameters.md)
- [Deref & DerefMut](../chapter-5/deref--derefmut.md)

```rust
trait AsRef<T: ?Sized> {
    fn as_ref(&self) -> &T;
}
trait AsMut<T: ?Sized> {
    fn as_mut(&mut self) -> &mut T;
}
```

`AsRef` is for cheap reference to reference conversions. However, one of the most common ways it's used is to make functions generic over whether they take ownership or not:

```rust
// accepts:
//  - &str
//  - &String
fn takes_str(s: &str) {
    // use &str
}
// accepts:
//  - &str
//  - &String
//  - String
fn takes_asref_str<S: AsRef<str>>(s: S) {
    let s: &str = s.as_ref();
    // use &str
}
fn example(slice: &str, borrow: &String, owned: String) {
    takes_str(slice);
    takes_str(borrow);
    takes_str(owned); // ❌
    takes_asref_str(slice);
    takes_asref_str(borrow);
    takes_asref_str(owned); // ✅
}
```

The other most common use-case is returning a reference to inner private data wrapped by a type which protects some invariant. A good example from the standard library is `String` which is just a wrapper around `Vec<u8>`:

```rust
struct String {
    vec: Vec<u8>,
}
```

This inner `Vec` cannot be made public because if it was people could mutate any byte and break the `String`'s valid UTF-8 encoding. However, it's safe to expose an immutable read-only reference to the inner byte array, hence this impl:

```rust
impl AsRef<[u8]> for String;
```

Generally, it often only makes sense to impl `AsRef` for a type if it wraps some other type to either provide additional functionality around the inner type or protect some invariant on the inner type.
Let's examine a example of bad `AsRef` impls:

```rust
struct User {
    name: String,
    age: u32,
}
impl AsRef<String> for User {
    fn as_ref(&self) -> &String {
        &self.name
    }
}
impl AsRef<u32> for User {
    fn as_ref(&self) -> &u32 {
        &self.age
    }
}
```

This works and kinda makes sense at first, but quickly falls apart if we add more members to `User`:

```rust
struct User {
    name: String,
    email: String,
    age: u32,
    height: u32,
}
impl AsRef<String> for User {
    fn as_ref(&self) -> &String {
        // uh, do we return name or email here?
    }
}
impl AsRef<u32> for User {
    fn as_ref(&self) -> &u32 {
        // uh, do we return age or height here?
    }
}
```

A `User` is composed of `String`s and `u32`s but it's not really the same thing as a `String` or a `u32`. Even if we had much more specific types:

```rust
struct User {
    name: Name,
    email: Email,
    age: Age,
    height: Height,
}
```

It wouldn't make much sense to impl `AsRef` for any of those because `AsRef` is for cheap reference to reference conversions between semantically equivalent things, and `Name`, `Email`, `Age`, and `Height` by themselves are not the same thing as a `User`.

A good example where we would impl `AsRef` would be if we introduced a new type `Moderator` that just wrapped a `User` and added some moderation specific privileges:

```rust
struct User {
    name: String,
    age: u32,
}
// unfortunately the standard library cannot provide
// a generic blanket impl to save us from this boilerplate
impl AsRef<User> for User {
    fn as_ref(&self) -> &User {
        self
    }
}
enum Privilege {
    BanUsers,
    EditPosts,
    DeletePosts,
}
// although Moderators have some special
// privileges they are still regular Users
// and should be able to do all the same stuff
struct Moderator {
    user: User,
    privileges: Vec<Privilege>
}
impl AsRef<Moderator> for Moderator {
    fn as_ref(&self) -> &Moderator {
        self
    }
}
impl AsRef<User> for Moderator {
    fn as_ref(&self) -> &User {
        &self.user
    }
}
// this should be callable with Users
// and Moderators (who are also Users)
fn create_post<U: AsRef<User>>(u: U) {
    let user = u.as_ref();
    // etc
}
fn example(user: User, moderator: Moderator) {
    create_post(&user);
    create_post(&moderator); // ✅
}
```

This works because `Moderator`s are just `User`s. Here's the example from the `Deref` section except using `AsRef` instead:

```rust
use std::convert::AsRef;
struct Human {
    health_points: u32,
}
impl AsRef<Human> for Human {
    fn as_ref(&self) -> &Human {
        self
    }
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
impl AsRef<Soldier> for Soldier {
    fn as_ref(&self) -> &Soldier {
        self
    }
}
impl AsRef<Human> for Soldier {
    fn as_ref(&self) -> &Human {
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
impl AsRef<Knight> for Knight {
    fn as_ref(&self) -> &Knight {
        self
    }
}
impl AsRef<Soldier> for Knight {
    fn as_ref(&self) -> &Soldier {
        &self.soldier
    }
}
impl AsRef<Human> for Knight {
    fn as_ref(&self) -> &Human {
        &self.soldier.human
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
impl AsRef<Mage> for Mage {
    fn as_ref(&self) -> &Mage {
        self
    }
}
impl AsRef<Human> for Mage {
    fn as_ref(&self) -> &Human {
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
impl AsRef<Wizard> for Wizard {
    fn as_ref(&self) -> &Wizard {
        self
    }
}
impl AsRef<Mage> for Wizard {
    fn as_ref(&self) -> &Mage {
        &self.mage
    }
}
impl AsRef<Human> for Wizard {
    fn as_ref(&self) -> &Human {
        &self.mage.human
    }
}
fn borrows_human<H: AsRef<Human>>(human: H) {}
fn borrows_soldier<S: AsRef<Soldier>>(soldier: S) {}
fn borrows_knight<K: AsRef<Knight>>(knight: K) {}
fn borrows_mage<M: AsRef<Mage>>(mage: M) {}
fn borrows_wizard<W: AsRef<Wizard>>(wizard: W) {}
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

`Deref` didn't work in the prior version of the example above because deref coercion is an implicit conversion between types which leaves room for people to mistakenly formulate the wrong ideas and expectations for how it will behave. `AsRef` works above because it makes the conversion between types explicit and there's no room leftover to develop any wrong ideas or expectations.

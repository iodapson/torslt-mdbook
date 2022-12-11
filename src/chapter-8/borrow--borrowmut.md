### Borrow & BorrowMut

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Generic Parameters](../chapter-1/generic-parameters.md)
- [Subtraits & Supertraits](../chapter-1/subtraits--supertraits.md)
- [Sized](../chapter-2/sized.md)
- [AsRef & AsMut](./asref--asmut.md)
- [PartialEq & Eq](../chapter-5/partialeq--eq.md)
- [Hash](../chapter-5/hash.md)
- [PartialOrd & Ord](../chapter-5/partialord--ord.md)

```rust
trait Borrow<Borrowed>
where
    Borrowed: ?Sized,
{
    fn borrow(&self) -> &Borrowed;
}
trait BorrowMut<Borrowed>: Borrow<Borrowed>
where
    Borrowed: ?Sized,
{
    fn borrow_mut(&mut self) -> &mut Borrowed;
}
```

These traits were invented to solve the very specific problem of looking up `String` keys in `HashSet`s, `HashMap`s, `BTreeSet`s, and `BTreeMap`s using `&str` values.

We can view `Borrow<T>` and `BorrowMut<T>` as stricter versions of `AsRef<T>` and `AsMut<T>`, where the returned reference `&T` has equivalent `Eq`, `Hash`, and `Ord` impls to `Self`. This is more easily explained with a commented example:

```rust
use std::borrow::Borrow;
use std::hash::Hasher;
use std::collections::hash_map::DefaultHasher;
use std::hash::Hash;
fn get_hash<T: Hash>(t: T) -> u64 {
    let mut hasher = DefaultHasher::new();
    t.hash(&mut hasher);
    hasher.finish()
}
fn asref_example<Owned, Ref>(owned1: Owned, owned2: Owned)
where
    Owned: Eq + Ord + Hash + AsRef<Ref>,
    Ref: Eq + Ord + Hash
{
    let ref1: &Ref = owned1.as_ref();
    let ref2: &Ref = owned2.as_ref();

    // refs aren't required to be equal if owned types are equal
    assert_eq!(owned1 == owned2, ref1 == ref2); // ❌

    let owned1_hash = get_hash(&owned1);
    let owned2_hash = get_hash(&owned2);
    let ref1_hash = get_hash(&ref1);
    let ref2_hash = get_hash(&ref2);

    // ref hashes aren't required to be equal if owned type hashes are equal
    assert_eq!(owned1_hash == owned2_hash, ref1_hash == ref2_hash); // ❌

    // ref comparisons aren't required to match owned type comparisons
    assert_eq!(owned1.cmp(&owned2), ref1.cmp(&ref2)); // ❌
}
fn borrow_example<Owned, Borrowed>(owned1: Owned, owned2: Owned)
where
    Owned: Eq + Ord + Hash + Borrow<Borrowed>,
    Borrowed: Eq + Ord + Hash
{
    let borrow1: &Borrowed = owned1.borrow();
    let borrow2: &Borrowed = owned2.borrow();

    // borrows are required to be equal if owned types are equal
    assert_eq!(owned1 == owned2, borrow1 == borrow2); // ✅

    let owned1_hash = get_hash(&owned1);
    let owned2_hash = get_hash(&owned2);
    let borrow1_hash = get_hash(&borrow1);
    let borrow2_hash = get_hash(&borrow2);

    // borrow hashes are required to be equal if owned type hashes are equal
    assert_eq!(owned1_hash == owned2_hash, borrow1_hash == borrow2_hash); // ✅

    // borrow comparisons are required to match owned type comparisons
    assert_eq!(owned1.cmp(&owned2), borrow1.cmp(&borrow2)); // ✅
}
```

It's good to be aware of these traits and understand why they exist since it helps demystify some of the methods on `HashSet`, `HashMap`, `BTreeSet`, and `BTreeMap` but it's very rare that we would ever need to impl these traits for any of our types because it's very rare that we would ever need create a pair of types where one is the "borrowed" version of the other in the first place. If we have some `T` then `&T` will get the job done 99.99% of the time, and `T: Borrow<T>` is already implemented for all `T` because of a generic blanket impl, so we don't need to manually impl it and we don't need to create some `U` such that `T: Borrow<U>`.

### IntoIterator

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Associated Types](../chapter-1/associated-types.md)
- [Iterator](./iterator.md)

```rust
trait IntoIterator
where
    <Self::IntoIter as Iterator>::Item == Self::Item,
{
    type Item;
    type IntoIter: Iterator;
    fn into_iter(self) -> Self::IntoIter;
}
```

`IntoIterator` types can be converted into iterators, hence the name. The `into_iter` method is called on a type when it's used within a `for-in` loop:

```rust
// vec = Vec<T>
for v in vec {} // v = T
// above line desugared
for v in vec.into_iter() {}
```

Not only does `Vec` impl `IntoIterator` but so does `&Vec` and `&mut Vec` if we'd like to iterate over immutable or mutable references instead of owned values, respectively.

```rust
// vec = Vec<T>
for v in &vec {} // v = &T
// above example desugared
for v in (&vec).into_iter() {}
// vec = Vec<T>
for v in &mut vec {} // v = &mut T
// above example desugared
for v in (&mut vec).into_iter() {}
```

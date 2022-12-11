### FromIterator

Prerequisites

- [Self](../chapter-1/self.md)
- [Functions](../chapter-1/functions.md)
- [Generic Parameters](../chapter-1/generic-parameters.md)
- [Iterator](./iterator.md)
- [IntoIterator](./intoiterator.md)

```rust
trait FromIterator<A> {
    fn from_iter<T>(iter: T) -> Self
    where
        T: IntoIterator<Item = A>;
}
```

`FromIterator` types can be created from an iterator, hence the name. `FromIterator` is most commonly and idiomatically used by calling the `collect` method on `Iterator`:

```rust
fn collect<B>(self) -> B
where
    B: FromIterator<Self::Item>;
```

Example of collecting an `Iterator<Item = char>` into a `String`:

```rust
fn filter_letters(string: &str) -> String {
    string.chars().filter(|c| c.is_alphabetic()).collect()
}
```

All the collections in the standard library impl `IntoIterator` and `FromIterator` so that makes it easier to convert between them:

```rust
use std::collections::{BTreeSet, HashMap, HashSet, LinkedList};
// String -> HashSet<char>
fn unique_chars(string: &str) -> HashSet<char> {
    string.chars().collect()
}
// Vec<T> -> BTreeSet<T>
fn ordered_unique_items<T: Ord>(vec: Vec<T>) -> BTreeSet<T> {
    vec.into_iter().collect()
}
// HashMap<K, V> -> LinkedList<(K, V)>
fn entry_list<K, V>(map: HashMap<K, V>) -> LinkedList<(K, V)> {
    map.into_iter().collect()
}
// and countless more possible examples
```

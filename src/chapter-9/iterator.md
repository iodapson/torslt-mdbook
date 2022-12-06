### Iterator

Prerequisites

- [Self](#self)
- [Methods](#methods)
- [Associated Types](#associated-types)
- [Default Impls](#default-impls)

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // provided default impls
    fn size_hint(&self) -> (usize, Option<usize>);
    fn count(self) -> usize;
    fn last(self) -> Option<Self::Item>;
    fn advance_by(&mut self, n: usize) -> Result<(), usize>;
    fn nth(&mut self, n: usize) -> Option<Self::Item>;
    fn step_by(self, step: usize) -> StepBy<Self>;
    fn chain<U>(
        self,
        other: U
    ) -> Chain<Self, <U as IntoIterator>::IntoIter>
    where
        U: IntoIterator<Item = Self::Item>;
    fn zip<U>(self, other: U) -> Zip<Self, <U as IntoIterator>::IntoIter>
    where
        U: IntoIterator;
    fn map<B, F>(self, f: F) -> Map<Self, F>
    where
        F: FnMut(Self::Item) -> B;
    fn for_each<F>(self, f: F)
    where
        F: FnMut(Self::Item);
    fn filter<P>(self, predicate: P) -> Filter<Self, P>
    where
        P: FnMut(&Self::Item) -> bool;
    fn filter_map<B, F>(self, f: F) -> FilterMap<Self, F>
    where
        F: FnMut(Self::Item) -> Option<B>;
    fn enumerate(self) -> Enumerate<Self>;
    fn peekable(self) -> Peekable<Self>;
    fn skip_while<P>(self, predicate: P) -> SkipWhile<Self, P>
    where
        P: FnMut(&Self::Item) -> bool;
    fn take_while<P>(self, predicate: P) -> TakeWhile<Self, P>
    where
        P: FnMut(&Self::Item) -> bool;
    fn map_while<B, P>(self, predicate: P) -> MapWhile<Self, P>
    where
        P: FnMut(Self::Item) -> Option<B>;
    fn skip(self, n: usize) -> Skip<Self>;
    fn take(self, n: usize) -> Take<Self>;
    fn scan<St, B, F>(self, initial_state: St, f: F) -> Scan<Self, St, F>
    where
        F: FnMut(&mut St, Self::Item) -> Option<B>;
    fn flat_map<U, F>(self, f: F) -> FlatMap<Self, U, F>
    where
        F: FnMut(Self::Item) -> U,
        U: IntoIterator;
    fn flatten(self) -> Flatten<Self>
    where
        Self::Item: IntoIterator;
    fn fuse(self) -> Fuse<Self>;
    fn inspect<F>(self, f: F) -> Inspect<Self, F>
    where
        F: FnMut(&Self::Item);
    fn by_ref(&mut self) -> &mut Self;
    fn collect<B>(self) -> B
    where
        B: FromIterator<Self::Item>;
    fn partition<B, F>(self, f: F) -> (B, B)
    where
        F: FnMut(&Self::Item) -> bool,
        B: Default + Extend<Self::Item>;
    fn partition_in_place<'a, T, P>(self, predicate: P) -> usize
    where
        Self: DoubleEndedIterator<Item = &'a mut T>,
        T: 'a,
        P: FnMut(&T) -> bool;
    fn is_partitioned<P>(self, predicate: P) -> bool
    where
        P: FnMut(Self::Item) -> bool;
    fn try_fold<B, F, R>(&mut self, init: B, f: F) -> R
    where
        F: FnMut(B, Self::Item) -> R,
        R: Try<Ok = B>;
    fn try_for_each<F, R>(&mut self, f: F) -> R
    where
        F: FnMut(Self::Item) -> R,
        R: Try<Ok = ()>;
    fn fold<B, F>(self, init: B, f: F) -> B
    where
        F: FnMut(B, Self::Item) -> B;
    fn fold_first<F>(self, f: F) -> Option<Self::Item>
    where
        F: FnMut(Self::Item, Self::Item) -> Self::Item;
    fn all<F>(&mut self, f: F) -> bool
    where
        F: FnMut(Self::Item) -> bool;
    fn any<F>(&mut self, f: F) -> bool
    where
        F: FnMut(Self::Item) -> bool;
    fn find<P>(&mut self, predicate: P) -> Option<Self::Item>
    where
        P: FnMut(&Self::Item) -> bool;
    fn find_map<B, F>(&mut self, f: F) -> Option<B>
    where
        F: FnMut(Self::Item) -> Option<B>;
    fn try_find<F, R>(
        &mut self,
        f: F
    ) -> Result<Option<Self::Item>, <R as Try>::Error>
    where
        F: FnMut(&Self::Item) -> R,
        R: Try<Ok = bool>;
    fn position<P>(&mut self, predicate: P) -> Option<usize>
    where
        P: FnMut(Self::Item) -> bool;
    fn rposition<P>(&mut self, predicate: P) -> Option<usize>
    where
        Self: ExactSizeIterator + DoubleEndedIterator,
        P: FnMut(Self::Item) -> bool;
    fn max(self) -> Option<Self::Item>
    where
        Self::Item: Ord;
    fn min(self) -> Option<Self::Item>
    where
        Self::Item: Ord;
    fn max_by_key<B, F>(self, f: F) -> Option<Self::Item>
    where
        F: FnMut(&Self::Item) -> B,
        B: Ord;
    fn max_by<F>(self, compare: F) -> Option<Self::Item>
    where
        F: FnMut(&Self::Item, &Self::Item) -> Ordering;
    fn min_by_key<B, F>(self, f: F) -> Option<Self::Item>
    where
        F: FnMut(&Self::Item) -> B,
        B: Ord;
    fn min_by<F>(self, compare: F) -> Option<Self::Item>
    where
        F: FnMut(&Self::Item, &Self::Item) -> Ordering;
    fn rev(self) -> Rev<Self>
    where
        Self: DoubleEndedIterator;
    fn unzip<A, B, FromA, FromB>(self) -> (FromA, FromB)
    where
        Self: Iterator<Item = (A, B)>,
        FromA: Default + Extend<A>,
        FromB: Default + Extend<B>;
    fn copied<'a, T>(self) -> Copied<Self>
    where
        Self: Iterator<Item = &'a T>,
        T: 'a + Copy;
    fn cloned<'a, T>(self) -> Cloned<Self>
    where
        Self: Iterator<Item = &'a T>,
        T: 'a + Clone;
    fn cycle(self) -> Cycle<Self>
    where
        Self: Clone;
    fn sum<S>(self) -> S
    where
        S: Sum<Self::Item>;
    fn product<P>(self) -> P
    where
        P: Product<Self::Item>;
    fn cmp<I>(self, other: I) -> Ordering
    where
        I: IntoIterator<Item = Self::Item>,
        Self::Item: Ord;
    fn cmp_by<I, F>(self, other: I, cmp: F) -> Ordering
    where
        F: FnMut(Self::Item, <I as IntoIterator>::Item) -> Ordering,
        I: IntoIterator;
    fn partial_cmp<I>(self, other: I) -> Option<Ordering>
    where
        I: IntoIterator,
        Self::Item: PartialOrd<<I as IntoIterator>::Item>;
    fn partial_cmp_by<I, F>(
        self,
        other: I,
        partial_cmp: F
    ) -> Option<Ordering>
    where
        F: FnMut(Self::Item, <I as IntoIterator>::Item) -> Option<Ordering>,
        I: IntoIterator;
    fn eq<I>(self, other: I) -> bool
    where
        I: IntoIterator,
        Self::Item: PartialEq<<I as IntoIterator>::Item>;
    fn eq_by<I, F>(self, other: I, eq: F) -> bool
    where
        F: FnMut(Self::Item, <I as IntoIterator>::Item) -> bool,
        I: IntoIterator;
    fn ne<I>(self, other: I) -> bool
    where
        I: IntoIterator,
        Self::Item: PartialEq<<I as IntoIterator>::Item>;
    fn lt<I>(self, other: I) -> bool
    where
        I: IntoIterator,
        Self::Item: PartialOrd<<I as IntoIterator>::Item>;
    fn le<I>(self, other: I) -> bool
    where
        I: IntoIterator,
        Self::Item: PartialOrd<<I as IntoIterator>::Item>;
    fn gt<I>(self, other: I) -> bool
    where
        I: IntoIterator,
        Self::Item: PartialOrd<<I as IntoIterator>::Item>;
    fn ge<I>(self, other: I) -> bool
    where
        I: IntoIterator,
        Self::Item: PartialOrd<<I as IntoIterator>::Item>;
    fn is_sorted(self) -> bool
    where
        Self::Item: PartialOrd<Self::Item>;
    fn is_sorted_by<F>(self, compare: F) -> bool
    where
        F: FnMut(&Self::Item, &Self::Item) -> Option<Ordering>;
    fn is_sorted_by_key<F, K>(self, f: F) -> bool
    where
        F: FnMut(Self::Item) -> K,
        K: PartialOrd<K>;
}
```

`Iterator<Item = T>` types can be iterated and will produce `T` types. There's no `IteratorMut` trait. Each `Iterator` impl can specify whether it returns immutable references, mutable references, or owned values via the `Item` associated type.

| `Vec<T>` method | Returns                   |
| --------------- | ------------------------- |
| `.iter()`       | `Iterator<Item = &T>`     |
| `.iter_mut()`   | `Iterator<Item = &mut T>` |
| `.into_iter()`  | `Iterator<Item = T>`      |

Something that is not immediately obvious to beginner Rustaceans but that intermediate Rustaceans take for granted is that most types are not their own iterators. If a type is iterable we almost always impl some custom iterator type which iterates over it rather than trying to make it iterate over itself:

```rust
struct MyType {
    items: Vec<String>
}
impl MyType {
    fn iter(&self) -> impl Iterator<Item = &String> {
        MyTypeIterator {
            index: 0,
            items: &self.items
        }
    }
}
struct MyTypeIterator<'a> {
    index: usize,
    items: &'a Vec<String>
}
impl<'a> Iterator for MyTypeIterator<'a> {
    type Item = &'a String;
    fn next(&mut self) -> Option<Self::Item> {
        if self.index >= self.items.len() {
            None
        } else {
            let item = &self.items[self.index];
            self.index += 1;
            Some(item)
        }
    }
}
```

For the sake of teaching the above example shows how to impl an `Iterator` from scratch but the idiomatic solution in this situation would be to just defer to `Vec`'s `iter` method:

```rust
struct MyType {
    items: Vec<String>
}
impl MyType {
    fn iter(&self) -> impl Iterator<Item = &String> {
        self.items.iter()
    }
}
```

Also this is a good generic blanket impl to be aware of:

```rust
impl<I: Iterator + ?Sized> Iterator for &mut I;
```

It says that any mutable reference to an iterator is also an iterator. This is useful to know because it allows us to use iterator methods with `self` receivers as if they had `&mut self` receivers.
As an example, imagine we have a function which processes an iterator of more than three items, but the first step of the function is to take out the first three items of the iterator and process them separately before iterating over the remaining items, here's how a beginner may attempt to write this function:

```rust
fn example<I: Iterator<Item = i32>>(mut iter: I) {
    let first3: Vec<i32> = iter.take(3).collect();
    for item in iter { // ❌ iter consumed in line above
        // process remaining items
    }
}
```

Well that's annoying. The `take` method has a `self` receiver so it seems like we cannot call it without consuming the whole iterator! Here's what a naive refactor of the above code might look like:

```rust
fn example<I: Iterator<Item = i32>>(mut iter: I) {
    let first3: Vec<i32> = vec![
        iter.next().unwrap(),
        iter.next().unwrap(),
        iter.next().unwrap(),
    ];
    for item in iter { // ✅
        // process remaining items
    }
}
```

Which is okay. However, the idiomatic refactor is actually:

```rust
fn example<I: Iterator<Item = i32>>(mut iter: I) {
    let first3: Vec<i32> = iter.by_ref().take(3).collect();
    for item in iter { // ✅
        // process remaining items
    }
}
```

Not very easy to discover. But anyway, now we know.

Also, there are no rules or conventions on what can or cannot be an iterator. If the type impls `Iterator` then it's an iterator. Some creative examples from the standard library:

```rust
use std::sync::mpsc::channel;
use std::thread;
fn paths_can_be_iterated(path: &Path) {
    for part in path {
        // iterate over parts of a path
    }
}
fn receivers_can_be_iterated() {
    let (send, recv) = channel();
    thread::spawn(move || {
        send.send(1).unwrap();
        send.send(2).unwrap();
        send.send(3).unwrap();
    });
    for received in recv {
        // iterate over received values
    }
}
```

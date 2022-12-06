#### FnOnce, FnMut, & Fn

Prerequisites

- [Self](#self)
- [Methods](#methods)
- [Associated Types](#associated-types)
- [Generic Parameters](#generic-parameters)
- [Generic Types vs Associated Types](#generic-types-vs-associated-types)
- [Subtraits & Supertraits](#subtraits--supertraits)

```rust
trait FnOnce<Args> {
    type Output;
    fn call_once(self, args: Args) -> Self::Output;
}
trait FnMut<Args>: FnOnce<Args> {
    fn call_mut(&mut self, args: Args) -> Self::Output;
}
trait Fn<Args>: FnMut<Args> {
    fn call(&self, args: Args) -> Self::Output;
}
```

Although these traits exist it's not possible to impl them for our own types in stable Rust. The only types we can create which impl these traits are closures. Depending on what the closure captures from its environment determines whether it impls `FnOnce`, `FnMut`, or `Fn`.

An `FnOnce` closure can only be called once because it consumes some value as part of its execution:

```rust
fn main() {
    let range = 0..10;
    let get_range_count = || range.count();
    assert_eq!(get_range_count(), 10); // ✅
    get_range_count(); // ❌
}
```

The `.count()` method on iterators consumes the iterator so it can only be called once. Hence our closure can only be called once. Which is why when we try to call it a second time we get this error:

```none
error[E0382]: use of moved value: `get_range_count`
 --> src/main.rs:5:5
  |
4 |     assert_eq!(get_range_count(), 10);
  |                ----------------- `get_range_count` moved due to this call
5 |     get_range_count();
  |     ^^^^^^^^^^^^^^^ value used here after move
  |
note: closure cannot be invoked more than once because it moves the variable `range` out of its environment
 --> src/main.rs:3:30
  |
3 |     let get_range_count = || range.count();
  |                              ^^^^^
note: this value implements `FnOnce`, which causes it to be moved when called
 --> src/main.rs:4:16
  |
4 |     assert_eq!(get_range_count(), 10);
  |                ^^^^^^^^^^^^^^^
```

An `FnMut` closure can be called multiple times and can also mutate variables it has captured from its environment. We might say `FnMut` closures perform side-effects or are stateful. Here's an example of a closure that filters out all non-ascending values from an iterator by keeping track of the smallest value it has seen so far:

```rust
fn main() {
    let nums = vec![0, 4, 2, 8, 10, 7, 15, 18, 13];
    let mut min = i32::MIN;
    let ascending = nums.into_iter().filter(|&n| {
        if n <= min {
            false
        } else {
            min = n;
            true
        }
    }).collect::<Vec<_>>();
    assert_eq!(vec![0, 4, 8, 10, 15, 18], ascending); // ✅
}
```

`FnMut` refines `FnOnce` in the sense that `FnOnce` requires taking ownership of its arguments and can only be called once, but `FnMut` requires only taking mutable references and can be called multiple times. `FnMut` can be used anywhere `FnOnce` can be used.

An `Fn` closure can be called multiple times and does not mutate any variables it has captured from its environment. We might say `Fn` closures have no side-effects or are stateless. Here's an example closure that filters out all values less than some stack variable it captures from its environment from an iterator:

```rust
fn main() {
    let nums = vec![0, 4, 2, 8, 10, 7, 15, 18, 13];
    let min = 9;
    let greater_than_9 = nums.into_iter().filter(|&n| n > min).collect::<Vec<_>>();
    assert_eq!(vec![10, 15, 18, 13], greater_than_9); // ✅
}
```

`Fn` refines `FnMut` in the sense that `FnMut` requires mutable references and can be called multiple times, but `Fn` only requires immutable references and can be called multiple times. `Fn` can be used anywhere `FnMut` can be used, which includes anywhere `FnOnce` can be used.

If a closure doesn't capture anything from its environment it's technically not a closure, but just an anonymously declared inline function, and can be casted to, used, and passed around as a regular function pointer, i.e. `fn`. Function pointers can be used anywhere `Fn` can be used, which includes anwhere `FnMut` and `FnOnce` can be used.

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
fn main() {
    let mut fn_ptr: fn(i32) -> i32 = add_one;
    assert_eq!(fn_ptr(1), 2); // ✅

    // capture-less closure cast to fn pointer
    fn_ptr = |x| x + 1; // same as add_one
    assert_eq!(fn_ptr(1), 2); // ✅
}
```

Example of passing a regular function pointer in place of a closure:

```rust
fn main() {
    let nums = vec![-1, 1, -2, 2, -3, 3];
    let absolutes: Vec<i32> = nums.into_iter().map(i32::abs).collect();
    assert_eq!(vec![1, 1, 2, 2, 3, 3], absolutes); // ✅
}
```

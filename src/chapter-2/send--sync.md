### Send & Sync

Prerequisites

- [Marker Traits](../chapter-1/marker-traits.md)
- [Auto Traits](../chapter-1/auto-traits.md)
- [Unsafe Traits](../chapter-1/unsafe-traits.md)

```rust
unsafe auto trait Send {}
unsafe auto trait Sync {}
```

If a type is `Send` that means it's safe to send between threads. If a type is `Sync` that means it's safe to share references of it between threads. In more precise terms some type `T` is `Sync` if and only if `&T` is `Send`.

Almost all types are `Send` and `Sync`. The only notable `Send` exception is `Rc` and the only notable `Sync` exceptions are `Rc`, `Cell`, `RefCell`. If we need a `Send` version of `Rc` we can use `Arc`. If we need a `Sync` version of `Cell` or `RefCell` we can `Mutex` or `RwLock`. Although if we're using the `Mutex` or `RwLock` to just wrap a primitive type it's often better to use the atomic primitive types provided by the standard library such as `AtomicBool`, `AtomicI32`, `AtomicUsize`, and so on.

That almost all types are `Sync` might be a surprise to some people, but yup, it's true even for types without any internal synchronization. This is possible thanks to Rust's strict borrowing rules.

We can pass many immutable references to the same data to many threads and we're guaranteed there are no data races because as long as any immutable references exist Rust statically guarantees the underlying data cannot be mutated:

```rust
use crossbeam::thread;
fn main() {
    let mut greeting = String::from("Hello");
    let greeting_ref = &greeting;

    thread::scope(|scoped_thread| {
        // spawn 3 threads
        for n in 1..=3 {
            // greeting_ref copied into every thread
            scoped_thread.spawn(move |_| {
                println!("{} {}", greeting_ref, n); // prints "Hello {n}"
            });
        }

        // line below could cause UB or data races but compiler rejects it
        greeting += " world"; // ❌ cannot mutate greeting while immutable refs exist
    });

    // can mutate greeting after every thread has joined
    greeting += " world"; // ✅
    println!("{}", greeting); // prints "Hello world"
}
```

Likewise we can pass a single mutable reference to some data to a single thread and we're guaranteed there will be no data races because Rust statically guarantees aliased mutable references cannot exist and the underlying data cannot be mutated through anything other than the single existing mutable reference:

```rust
use crossbeam::thread;
fn main() {
    let mut greeting = String::from("Hello");
    let greeting_ref = &mut greeting;

    thread::scope(|scoped_thread| {
        // greeting_ref moved into thread
        scoped_thread.spawn(move |_| {
            *greeting_ref += " world";
            println!("{}", greeting_ref); // prints "Hello world"
        });

        // line below could cause UB or data races but compiler rejects it
        greeting += "!!!"; // ❌ cannot mutate greeting while mutable refs exist
    });

    // can mutate greeting after the thread has joined
    greeting += "!!!"; // ✅
    println!("{}", greeting); // prints "Hello world!!!"
}
```

This is why most types are `Sync` without requiring any explicit synchronization. In the event we need to simultaneously mutate some data `T` across multiple threads the compiler won't let us until we wrap the data in a `Arc<Mutex<T>>` or `Arc<RwLock<T>>` so the compiler enforces that explicit synchronization is used when it's needed.

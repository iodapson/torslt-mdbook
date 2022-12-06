### Scope

Trait items cannot be used unless the trait is in scope. Most Rustaceans learn this the hard way the first time they try to write a program that does anything with I/O because the `Read` and `Write` traits are not in the standard library prelude:

```rust
use std::fs::File;
use std::io;
fn main() -> Result<(), io::Error> {
    let mut file = File::open("Cargo.toml")?;
    let mut buffer = String::new();
    file.read_to_string(&mut buffer)?; // ❌ read_to_string not found in File
    Ok(())
}
```

`read_to_string(buf: &mut String)` is declared by the `std::io::Read` trait and implemented by the `std::fs::File` struct but in order to call it `std::io::Read` must be in scope:

```rust
use std::fs::File;
use std::io;
use std::io::Read; // ✅
fn main() -> Result<(), io::Error> {
    let mut file = File::open("Cargo.toml")?;
    let mut buffer = String::new();
    file.read_to_string(&mut buffer)?; // ✅
    Ok(())
}
```

The standard library prelude is a module in the standard library, i.e. `std::prelude::v1`, that gets auto imported at the top of every other module, i.e. `use std::prelude::v1::*`. Thus the following traits are always in scope and we never have to explicitly import them ourselves because they're part of the prelude:

- [AsMut](../chapter-8/asref--asmut.md)
- [AsRef](../chapter-8/asref--asmut.md)
- [Clone](../chapter-3/clone.md)
- [Copy](../chapter-3/copy.md)
- [Default](../chapter-3/default.md)
- [Drop](../chapter-5/drop.md)
- [Eq](../chapter-5/partialeq--eq.md)
- [Fn](../chapter-5/fnonce-fnmut--fn.md)
- [FnMut](../chapter-5/fnonce-fnmut--fn.md)
- [FnOnce](../chapter-5/fnonce-fnmut--fn.md)
- [From](../chapter-6/from--into.md)
- [Into](../chapter-6/from--into.md)
- [ToOwned](../chapter-8/toowned.md)
- [IntoIterator](../chapter-9/intoiterator.md)
- [Iterator](../chapter-9/iterator.md)
- [PartialEq](../chapter-5/partialeq--eq.md)
- [PartialOrd](../chapter-5/partialord--ord.md)
- [Send](../chapter-2/send--sync.md)
- [Sized](../chapter-2/sized.md)
- [Sync](../chapter-2/send--sync.md)
- [ToString](../chapter-4/display--tostring.md)
- [Ord](../chapter-5/partialord--ord.md)

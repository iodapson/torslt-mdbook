## Error Handling

The best time to talk about error handling and the `Error` trait is after going over `Display`, `Debug`, `Any`, and `From` but before getting to `TryFrom` hence why the **Error Handling** section awkwardly bisects the **Conversion Traits** section.

### Error

Prerequisites

- [Self](../chapter-1/self.md)
- [Methods](../chapter-1/methods.md)
- [Default Impls](../chapter-1/default-impls.md)
- [Generic Blanket Impls](../chapter-1/generic-blanket-impls.md)
- [Subtraits & Supertraits](../chapter-1/subtraits--supertraits.md)
- [Trait Objects](../chapter-1/trait-objects.md)
- [Display & ToString](../chapter-4/display--tostring.md)
- [Debug](../chapter-4/debug.md)
- [Any](../chapter-3/any.md)
- [From & Into](../chapter-6/from--into.md)

```rust
trait Error: Debug + Display {
    // provided default impls
    fn source(&self) -> Option<&(dyn Error + 'static)>;
    fn backtrace(&self) -> Option<&Backtrace>;
    fn description(&self) -> &str;
    fn cause(&self) -> Option<&dyn Error>;
}
```

In Rust errors are returned, not thrown. Let's look at some examples.

Since dividing integer types by zero panics if we wanted to make our program safer and more explicit we could impl a `safe_div` function which returns a `Result` instead like this:

```rust
use std::fmt;
use std::error;
#[derive(Debug, PartialEq)]
struct DivByZero;
impl fmt::Display for DivByZero {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "division by zero error")
    }
}
impl error::Error for DivByZero {}
fn safe_div(numerator: i32, denominator: i32) -> Result<i32, DivByZero> {
    if denominator == 0 {
        return Err(DivByZero);
    }
    Ok(numerator / denominator)
}
#[test] // ✅
fn test_safe_div() {
    assert_eq!(safe_div(8, 2), Ok(4));
    assert_eq!(safe_div(5, 0), Err(DivByZero));
}
```

Since errors are returned and not thrown they must be explicitly handled, and if the current function cannot handle an error it should propagate it up to the caller. The most idiomatic way to propagate errors is to use the `?` operator, which is just syntax sugar for the now deprecated `try!` macro which simply does this:

```rust
macro_rules! try {
    ($expr:expr) => {
        match $expr {
            // if Ok just unwrap the value
            Ok(val) => val,
            // if Err map the err value using From and return
            Err(err) => {
                return Err(From::from(err));
            }
        }
    };
}
```

If we wanted to write a function which reads a file into a `String` we could write it like this, propagating the `io::Error`s using `?` everywhere they can appear:

```rust
use std::io::Read;
use std::path::Path;
use std::io;
use std::fs::File;
fn read_file_to_string(path: &Path) -> Result<String, io::Error> {
    let mut file = File::open(path)?; // ⬆️ io::Error
    let mut contents = String::new();
    file.read_to_string(&mut contents)?; // ⬆️ io::Error
    Ok(contents)
}
```

But let's say the file we're reading is actually a list of numbers and we want to sum them together, we'd update our function like this:

```rust
use std::io::Read;
use std::path::Path;
use std::io;
use std::fs::File;
fn sum_file(path: &Path) -> Result<i32, /* What to put here? */> {
    let mut file = File::open(path)?; // ⬆️ io::Error
    let mut contents = String::new();
    file.read_to_string(&mut contents)?; // ⬆️ io::Error
    let mut sum = 0;
    for line in contents.lines() {
        sum += line.parse::<i32>()?; // ⬆️ ParseIntError
    }
    Ok(sum)
}
```

But what's the error type of our `Result` now? It can return either an `io::Error` or a `ParseIntError`. We're going to look at three approaches for solving this problem, starting with the most quick & dirty way and finishing with the most robust way.

The first approach is recognizing that all types which impl `Error` also impl `Display` so we can map all the errors to `String`s and use `String` as our error type:

```rust
use std::fs::File;
use std::io;
use std::io::Read;
use std::path::Path;
fn sum_file(path: &Path) -> Result<i32, String> {
    let mut file = File::open(path)
        .map_err(|e| e.to_string())?; // ⬆️ io::Error -> String
    let mut contents = String::new();
    file.read_to_string(&mut contents)
        .map_err(|e| e.to_string())?; // ⬆️ io::Error -> String
    let mut sum = 0;
    for line in contents.lines() {
        sum += line.parse::<i32>()
            .map_err(|e| e.to_string())?; // ⬆️ ParseIntError -> String
    }
    Ok(sum)
}
```

The obvious downside of stringifying every error is that we throw away type information which makes it harder for the caller to handle the errors.

One nonobvious upside to the above approach is we can customize the strings to provide more context-specific information. For example, `ParseIntError` usually stringifies to `"invalid digit found in string"` which is very vague and doesn't mention what the invalid string is or what integer type it was trying to parse into. If we were debugging this problem that error message would almost be useless. However we can make it significantly better by providing all the context relevant information ourselves:

```rust
sum += line.parse::<i32>()
    .map_err(|_| format!("failed to parse {} into i32", line))?;
```

The second approach takes advantage of this generic blanket impl from the standard library:

```rust
impl<E: error::Error> From<E> for Box<dyn error::Error>;
```

Which means that any `Error` type can be implicitly converted into a `Box<dyn error::Error>` by the `?` operator, so we can set to error type to `Box<dyn error::Error>` in the `Result` return type of any function which produces errors and the `?` operator will do the rest of the work for us:

```rust
use std::fs::File;
use std::io::Read;
use std::path::Path;
use std::error;
fn sum_file(path: &Path) -> Result<i32, Box<dyn error::Error>> {
    let mut file = File::open(path)?; // ⬆️ io::Error -> Box<dyn error::Error>
    let mut contents = String::new();
    file.read_to_string(&mut contents)?; // ⬆️ io::Error -> Box<dyn error::Error>
    let mut sum = 0;
    for line in contents.lines() {
        sum += line.parse::<i32>()?; // ⬆️ ParseIntError -> Box<dyn error::Error>
    }
    Ok(sum)
}
```

While being more concise, this seems to suffer from the same downside of the previous approach by throwing away type information. This is mostly true, but if the caller is aware of the impl details of our function they can still handle the different errors types using the `downcast_ref()` method on `error::Error` which works the same as it does on `dyn Any` types:

```rust
fn handle_sum_file_errors(path: &Path) {
    match sum_file(path) {
        Ok(sum) => println!("the sum is {}", sum),
        Err(err) => {
            if let Some(e) = err.downcast_ref::<io::Error>() {
                // handle io::Error
            } else if let Some(e) = err.downcast_ref::<ParseIntError>() {
                // handle ParseIntError
            } else {
                // we know sum_file can only return one of the
                // above errors so this branch is unreachable
                unreachable!();
            }
        }
    }
}
```

The third approach, which is the most robust and type-safe way to aggregate these different errors would be to build our own custom error type using an enum:

```rust
use std::num::ParseIntError;
use std::fs::File;
use std::io;
use std::io::Read;
use std::path::Path;
use std::error;
use std::fmt;
#[derive(Debug)]
enum SumFileError {
    Io(io::Error),
    Parse(ParseIntError),
}
impl From<io::Error> for SumFileError {
    fn from(err: io::Error) -> Self {
        SumFileError::Io(err)
    }
}
impl From<ParseIntError> for SumFileError {
    fn from(err: ParseIntError) -> Self {
        SumFileError::Parse(err)
    }
}
impl fmt::Display for SumFileError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            SumFileError::Io(err) => write!(f, "sum file error: {}", err),
            SumFileError::Parse(err) => write!(f, "sum file error: {}", err),
        }
    }
}
impl error::Error for SumFileError {
    // the default impl for this method always returns None
    // but we can now override it to make it way more useful!
    fn source(&self) -> Option<&(dyn error::Error + 'static)> {
        Some(match self {
            SumFileError::Io(err) => err,
            SumFileError::Parse(err) => err,
        })
    }
}
fn sum_file(path: &Path) -> Result<i32, SumFileError> {
    let mut file = File::open(path)?; // ⬆️ io::Error -> SumFileError
    let mut contents = String::new();
    file.read_to_string(&mut contents)?; // ⬆️ io::Error -> SumFileError
    let mut sum = 0;
    for line in contents.lines() {
        sum += line.parse::<i32>()?; // ⬆️ ParseIntError -> SumFileError
    }
    Ok(sum)
}
fn handle_sum_file_errors(path: &Path) {
    match sum_file(path) {
        Ok(sum) => println!("the sum is {}", sum),
        Err(SumFileError::Io(err)) => {
            // handle io::Error
        },
        Err(SumFileError::Parse(err)) => {
            // handle ParseIntError
        },
    }
}
```

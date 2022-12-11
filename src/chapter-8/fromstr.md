### FromStr

Prerequisites

- [Self](../chapter-1/self.md)
- [Functions](../chapter-1/functions.md)
- [Associated Types](../chapter-1/associated-types.md)
- [Error](../chapter-7/error.md)
- [TryFrom & TryInto](../chapter-8/tryfrom--tryinto.md)

```rust
trait FromStr {
    type Err;
    fn from_str(s: &str) -> Result<Self, Self::Err>;
}
```

`FromStr` types allow performing a fallible conversion from `&str` into `Self`. The idiomatic way to use `FromStr` is to call the `.parse()` method on `&str`s:

```rust
use std::str::FromStr;
fn example<T: FromStr>(s: &'static str) {
    // these are all equivalent
    let t: Result<T, _> = FromStr::from_str(s);
    let t = T::from_str(s);
    let t: Result<T, _> = s.parse();
    let t = s.parse::<T>(); // most idiomatic
}
```

Example impl for `Point`:

```rust
use std::error;
use std::fmt;
use std::iter::Enumerate;
use std::num::ParseIntError;
use std::str::{Chars, FromStr};
#[derive(Debug, Eq, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
impl Point {
    fn new(x: i32, y: i32) -> Self {
        Point { x, y }
    }
}
#[derive(Debug, PartialEq)]
struct ParsePointError;
impl fmt::Display for ParsePointError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "failed to parse point")
    }
}
impl From<ParseIntError> for ParsePointError {
    fn from(_e: ParseIntError) -> Self {
        ParsePointError
    }
}
impl error::Error for ParsePointError {}
impl FromStr for Point {
    type Err = ParsePointError;
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let is_num = |(_, c): &(usize, char)| matches!(c, '0'..='9' | '-');
        let isnt_num = |t: &(_, _)| !is_num(t);
        let get_num =
            |char_idxs: &mut Enumerate<Chars<'_>>| -> Result<(usize, usize), ParsePointError> {
                let (start, _) = char_idxs
                    .skip_while(isnt_num)
                    .next()
                    .ok_or(ParsePointError)?;
                let (end, _) = char_idxs
                    .skip_while(is_num)
                    .next()
                    .ok_or(ParsePointError)?;
                Ok((start, end))
            };
        let mut char_idxs = s.chars().enumerate();
        let (x_start, x_end) = get_num(&mut char_idxs)?;
        let (y_start, y_end) = get_num(&mut char_idxs)?;
        let x = s[x_start..x_end].parse::<i32>()?;
        let y = s[y_start..y_end].parse::<i32>()?;
        Ok(Point { x, y })
    }
}
#[test] // ✅
fn pos_x_y() {
    let p = "(4, 5)".parse::<Point>();
    assert_eq!(p, Ok(Point::new(4, 5)));
}
#[test] // ✅
fn neg_x_y() {
    let p = "(-6, -2)".parse::<Point>();
    assert_eq!(p, Ok(Point::new(-6, -2)));
}
#[test] // ✅
fn not_a_point() {
    let p = "not a point".parse::<Point>();
    assert_eq!(p, Err(ParsePointError));
}
```

`FromStr` has the same signature as `TryFrom<&str>`. It doesn't matter which one we impl for a type first as long as we forward the impl to the other one. Here's a `TryFrom<&str>` impl for `Point` assuming it already has a `FromStr` impl:

```rust
impl TryFrom<&str> for Point {
    type Error = <Point as FromStr>::Err;
    fn try_from(s: &str) -> Result<Point, Self::Error> {
        <Point as FromStr>::from_str(s)
    }
}
```

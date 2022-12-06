## Operator Traits

All operators in Rust are associated with traits. If we'd like to impl operators for our types we have to impl the associated traits.

| Trait(s)            | Category   | Operator(s)          | Description                  |
| ------------------- | ---------- | -------------------- | ---------------------------- |
| `Eq`, `PartialEq`   | comparison | `==`                 | equality                     |
| `Ord`, `PartialOrd` | comparison | `<`, `>`, `<=`, `>=` | comparison                   |
| `Add`               | arithmetic | `+`                  | addition                     |
| `AddAssign`         | arithmetic | `+=`                 | addition assignment          |
| `BitAnd`            | arithmetic | `&`                  | bitwise AND                  |
| `BitAndAssign`      | arithmetic | `&=`                 | bitwise assignment           |
| `BitXor`            | arithmetic | `^`                  | bitwise XOR                  |
| `BitXorAssign`      | arithmetic | `^=`                 | bitwise XOR assignment       |
| `Div`               | arithmetic | `/`                  | division                     |
| `DivAssign`         | arithmetic | `/=`                 | division assignment          |
| `Mul`               | arithmetic | `*`                  | multiplication               |
| `MulAssign`         | arithmetic | `*=`                 | multiplication assignment    |
| `Neg`               | arithmetic | `-`                  | unary negation               |
| `Not`               | arithmetic | `!`                  | unary logical negation       |
| `Rem`               | arithmetic | `%`                  | remainder                    |
| `RemAssign`         | arithmetic | `%=`                 | remainder assignment         |
| `Shl`               | arithmetic | `<<`                 | left shift                   |
| `ShlAssign`         | arithmetic | `<<=`                | left shift assignment        |
| `Shr`               | arithmetic | `>>`                 | right shift                  |
| `ShrAssign`         | arithmetic | `>>=`                | right shift assignment       |
| `Sub`               | arithmetic | `-`                  | subtraction                  |
| `SubAssign`         | arithmetic | `-=`                 | subtraction assignment       |
| `Fn`                | closure    | `(...args)`          | immutable closure invocation |
| `FnMut`             | closure    | `(...args)`          | mutable closure invocation   |
| `FnOnce`            | closure    | `(...args)`          | one-time closure invocation  |
| `Deref`             | other      | `*`                  | immutable dereference        |
| `DerefMut`          | other      | `*`                  | mutable dereference          |
| `Drop`              | other      | -                    | type destructor              |
| `Index`             | other      | `[]`                 | immutable index              |
| `IndexMut`          | other      | `[]`                 | mutable index                |
| `RangeBounds`       | other      | `..`                 | range                        |

### Arithmetic Traits

| Trait(s)       | Category   | Operator(s) | Description               |
| -------------- | ---------- | ----------- | ------------------------- |
| `Add`          | arithmetic | `+`         | addition                  |
| `AddAssign`    | arithmetic | `+=`        | addition assignment       |
| `BitAnd`       | arithmetic | `&`         | bitwise AND               |
| `BitAndAssign` | arithmetic | `&=`        | bitwise assignment        |
| `BitXor`       | arithmetic | `^`         | bitwise XOR               |
| `BitXorAssign` | arithmetic | `^=`        | bitwise XOR assignment    |
| `Div`          | arithmetic | `/`         | division                  |
| `DivAssign`    | arithmetic | `/=`        | division assignment       |
| `Mul`          | arithmetic | `*`         | multiplication            |
| `MulAssign`    | arithmetic | `*=`        | multiplication assignment |
| `Neg`          | arithmetic | `-`         | unary negation            |
| `Not`          | arithmetic | `!`         | unary logical negation    |
| `Rem`          | arithmetic | `%`         | remainder                 |
| `RemAssign`    | arithmetic | `%=`        | remainder assignment      |
| `Shl`          | arithmetic | `<<`        | left shift                |
| `ShlAssign`    | arithmetic | `<<=`       | left shift assignment     |
| `Shr`          | arithmetic | `>>`        | right shift               |
| `ShrAssign`    | arithmetic | `>>=`       | right shift assignment    |
| `Sub`          | arithmetic | `-`         | subtraction               |
| `SubAssign`    | arithmetic | `-=`        | subtraction assignment    |

Going over all of these would be very redundant. Most of these only apply to number types anyway. We'll only go over `Add` and `AddAssign` since the `+` operator is commonly overloaded to do other stuff like adding items to collections or concatenating things together, that way we cover the most interesting ground and don't repeat ourselves.

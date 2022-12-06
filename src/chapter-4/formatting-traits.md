We can serialize types into strings using the formatting macros in `std::fmt`, the most well-known of the bunch being `println!`. We can pass formatting parameters to the `{}` placeholders used within format `str`s which are then used to select which trait impl to use to serialize the placeholder's argument.

| Trait      | Placeholder | Description                          |
| ---------- | ----------- | ------------------------------------ |
| `Display`  | `{}`        | display representation               |
| `Debug`    | `{:?}`      | debug representation                 |
| `Octal`    | `{:o}`      | octal representation                 |
| `LowerHex` | `{:x}`      | lowercase hex representation         |
| `UpperHex` | `{:X}`      | uppercase hex representation         |
| `Pointer`  | `{:p}`      | memory address                       |
| `Binary`   | `{:b}`      | binary representation                |
| `LowerExp` | `{:e}`      | lowercase exponential representation |
| `UpperExp` | `{:E}`      | uppercase exponential representation |

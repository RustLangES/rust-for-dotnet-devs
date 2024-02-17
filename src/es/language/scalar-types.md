# Tipos Escalares

La siguiente tabla enumera los tipos primitivos en Rust y su equivalente en
C# y .NET:

| Rust    | C#        | .NET                   | Notas                  |
| ------- | --------- | ---------------------- | -----------------------|
| `bool`  | `bool`    | `Boolean`              |                        |
| `char`  | `char`    | `Char`                 | Mirar la nota 1.       |
| `i8`    | `sbyte`   | `SByte`                |                        |
| `i16`   | `short`   | `Int16`                |                        |
| `i32`   | `int`     | `Int32`                |                        |
| `i64`   | `long`    | `Int64`                |                        |
| `i128`  |           | `Int128`               |                        |
| `isize` | `nint`    | `IntPtr`               |                        |
| `u8`    | `byte`    | `Byte`                 |                        |
| `u16`   | `ushort`  | `UInt16`               |                        |
| `u32`   | `uint`    | `UInt32`               |                        |
| `u64`   | `ulong`   | `UInt64`               |                        |
| `u128`  |           | `UInt128`              |                        |
| `usize` | `nuint`   | `UIntPtr`              |                        |
| `f32`   | `float`   | `Single`               |                        |
| `f64`   | `double`  | `Double`               |                        |
|         | `decimal` | `Decimal`              |                        |
| `()`    | `void`    | `Void` o `ValueTuple`  | Mirar las notas 2 y 3. |
|         | `object`  | `Object`               | Mirar la nota 3.       |

Notas:

1. [`char`][char.rs] en Rust y [`Char`][char.net] en .NET tienen diferentes
   definiciones. En Rust, un `char` tiene 4 bytes de ancho y es un [Unicode 
   scalar value], pero en .NET, a `Char` tiene 2 bytes de ancho y almacena el 
   carácter usando la codificación UTF-16. Para más información, mirar la [documentación de `char` en Rust][char.rs].

2. Mientras que en Rust, unit `()` (una tupla vacía) es un _valor expresable_, 
   el equivalente más cercano en C# sería `void` para representar la nada. 
   Sin embargo, `void` no es un _valor expresable_, excepto cuando se usan 
   punteros y código no seguro. .NET tiene [`ValueTuple`][ValueTuple], que es 
   una tupla vacía, pero C# no tiene una sintaxis literal como `()` para 
   representarlo. `ValueTuple` se puede usar en C#, pero es muy poco común. A 
   diferencia de C#, [F# sí tiene un tipo unit][unit.fs] similar a Rust.

3. Mientras `void` y `object` no son tipos escalares (aunque tipos escalares 
   como `int` son subclases de `object` en la jerarquía de tipos de .NET), se 
   han incluido en la tabla anterior por conveniencia.

Mira también:

- [Primitivos (Rust By Example)][primitives.rs]

[char.net]: https://learn.microsoft.com/en-us/dotnet/api/system.char
[char.rs]: https://doc.rust-lang.org/std/primitive.char.html
[Unicode scalar value]: https://www.unicode.org/glossary/#unicode_scalar_value
[ValueTuple]: https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple?view=net-7.0
[unit.fs]: https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/unit-type
[primitives.rs]: https://doc.rust-lang.org/rust-by-example/primitives.html

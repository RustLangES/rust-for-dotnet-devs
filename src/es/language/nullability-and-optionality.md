# Nulabilidad y Opcionalidad

En C#, `null` es a veces usado para representar un valor que es faltante, ausente
o lógicamente no inicializado. Por ejemplo:

```csharp
int? some = 1;
int? none = null;
```

Rust no tiene `null` y consecuencialmente contexto no nulleable para habilitar.
Los valores opcionales o faltantes son representados por [`Option<T>`][option]
en su lugar. El equivalente de el código C# de arriba en Rust debería ser:

```rust
let some: Option<i32> = Some(1);
let none: Option<i32> = None;
```

`Option<T>` en Rust es prácticamente idéntico a [`'T option`][opt.fs] de F#.

[opt.fs]: https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-option-1.html

## Flujo de Control con Opcionabilidad

En C#, tal vez estes usando sentencias `if`/`else` para controlar el flujo 
cuando uses valores nulleables.


```csharp
uint? max = 10;
if (max is { } someMax)
{
    Console.WriteLine($"El máximo es {someMax}."); // El máximo es 10.
}
```

Tu puedes usar pattern matching para lograr el mismo comportamiento en Rust:

Sería más conciso usar `if let`:

```rust
let max = Some(10u32);
if let Some(max) = max {
    println!("El máximo es {}.", max); // El máximo es 10.
}
```

## Operadores de Condición Nula

Los operadores null-condicionales (`?.` y `?[]`) facilitan el manejo de null en 
C#. En Rust, es mejor reemplazarlos usando el método [`map`][optmap]. El siguiente
fragmento muestra la comparación:

```csharp
string? some = "Hola, Mundo!";
string? none = null;
Console.WriteLine(some?.Length); // 12
Console.WriteLine(none?.Length); // (blank)
```

```rust
let some: Option<String> = Some(String::from("Hola, Mundo!"));
let none: Option<String> = None;
println!("{:?}", some.map(|s| s.len())); // Some(12)
println!("{:?}", none.map(|s| s.len())); // None
```

## Null-coalescing operator

El null-coalescing operator (`??`) es típicamente usado para por defecto usar 
otro valor cuando un nulleable es `null`:

```csharp
int? some = 1;
int? none = null;
Console.WriteLine(some ?? 0); // 1
Console.WriteLine(none ?? 0); // 0
```

En Rust, tu puedes usar [`unwrap_or`][unwrap-or] para obtener el mismo 
comportamiento:

```rust
let some: Option<i32> = Some(1);
let none: Option<i32> = None;
println!("{:?}", some.unwrap_or(0)); // 1
println!("{:?}", none.unwrap_or(0)); // 0
```

**Nota**: Si el valor por defecto es costoso para computar, tu puedes usar 
`unwrap_or_else` en su lugar. Este tomara una [closure] como argumento, la cual
permitirá inicializar un valor por defecto de forma [perezosa].

## Null-forgiving operator

El operador null-forgiving (`!`) no corresponde a un equivalente construido en 
Rust, como este solo afecta al flujo de análisis estático en el compilador de C#.
En Rust, esto no es necesario de usar para un sustituto de este.

[option]: https://doc.rust-lang.org/std/option/enum.Option.html
[optmap]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[unwrap-or]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
[perezosa]: https://es.wikipedia.org/wiki/Evaluación_perezosa
[closure]: ../language/lambda-and-closures
# Descartes

En C#, los [descartes][net-discards] expresan al compilador y a otros que 
ignoren los resultados (o partes) de una expresión.

Hay múltiples contextos donde se pueden aplicar, por ejemplo, como un ejemplo 
básico, para ignorar el resultado de una expresión. En C#, se vería así:

```csharp
_ = city.GetCityInformation(cityName);
```

En Rust, [ignorar el resultado de una expresión][rust-ignoring-values] se ve de 
manera idéntica:

```rust
_ = city.get_city_information(city_name);
```

Los descartes también se aplican para la destrucción de tuplas en C#:

```csharp
var (_, second) = ("first", "second");
```

y, de manera idéntica, en Rust:

```rust
let (_, second) = ("first", "second");
```

Además de la destrucción de tuplas, Rust ofrece [desestructuración][rust-destructuring]
de estructuras y enumeraciones usando `..`, donde `..` representa la parte 
restante de un tipo:

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x), // x is 0
}
```

Cuando se realiza pattern matching, a menudo es útil descartar o ignorar parte 
de una expresión coincidente, por ejemplo, en C#:

```csharp
_ = ("first", "second") switch
{
    ("first", _) => "first element matched",
    (_, _) => "first element did not match"
};
```

Y nuevamente, esto se ve casi idéntico en Rust:

```rust
_ = match ("first", "second")
{
    ("first", _) => "first element matched",
    (_, _) => "first element did not match"
};
```

[net-discards]: https://learn.microsoft.com/es-es/dotnet/csharp/fundamentals/functional/discards
[rust-ignoring-values]: https://book.rustlang-es.org/ch18-03-pattern-syntax?highlight=ignorando%20valore#ignorando-valores-en-un-pattern
[rust-destructuring]: https://doc.rust-lang.org/reference/patterns.html#destructuring

# Variables

Considera el siguiente ejemplo acerca de asignación de variables en C#:

```csharp
int x = 5;
```

Y el mismo en Rust:

```rust
let x: i32 = 5;
```

Hasta este momento, la única diferencia visible entre los dos lenguajes es que
la posición de la declaración del tipo es diferente. También, ambos, C# y Rust
son type-safe: el compilador garantiza que los valores almacenados en una
variable tiene siempre el mismo tipo designado. El ejemplo puede ser 
simplificado por usar la habilidad del compilador para automáticamente inferir 
el tipo de la variable. En C#:

```csharp
var x = 5;
```

En Rust:

```rust
let x = 5;
```

Cuando ampliamos el primer ejemplo para cambiar el valor de la variable 
(reasignamiento), el comportamiento de C# y Rust difieren:

```csharp
var x = 5;
x = 6;
Console.WriteLine(x); // 6
```

En Rust, la misma sentencia no compila:

```rust
let x = 5;
x = 6; // Error: cannot assign twice to immutable variable 'x'.
println!("{}", x);
```

En Rust, las variables son _inmutables_ por defecto. Una vez que un valor es
vinculado a un nombre, la variable no puede ser cambiada. Las variables pueden 
ser _mutables_ por adición de [`mut`][mut.rs] al comienzo de la declaración.

```rust
let mut x = 5;
x = 6;
println!("{}", x); // 6
```

Rust ofrece una alternativa para arreglar este ejemplo de encima que no requiere
mutabilidad mediante _shadowing_:

```rust
let x = 5;
let x = 6;
println!("{}", x); // 6
```

C# también soporta shadowing, por ejemplo variables locales pueden ocultar 
campos y variables miembros del tipo base. En Rust, el ejemplo de arriba 
demuestra que el shadowing también permite cambiar el tipo sin cambiar el nombre,
esto puede ser util si solo queremos transformar el dato en uno con diferente 
tipo y forma sin tener que tener una variable con distinto nombre en cada ocasión.

Puedes ver también:

- [Data races y race conditions] para más información acerca de las 
  implicaciones de la mutabilidad 
- [Scope y shadowing]
- [Memory management][memory-management-section] para explicaciones acerca de
  _moving_ y _ownership_

[mut.rs]: https://doc.rust-lang.org/std/keyword.mut.html
[memory-management-section]: ../memory-management/index.md
[data races y race conditions]: https://doc.rust-lang.org/nomicon/races.html
[scope y shadowing]: https://doc.rust-lang.org/stable/rust-by-example/variable_bindings/scope.html#scope-and-shadowing

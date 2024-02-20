# Estructuras (`struct`)

Las estructuras en Rust y C# comparten algunas similitudes:

- Se definen con la palabra clave `struct`, pero en Rust, `struct` simplemente 
  define los datos/campos. Los aspectos de comportamiento en términos de 
  funciones y métodos se definen por separado en un 
  _bloque de implementación_ (`impl`).

- Pueden implementar múltiples traits en Rust de la misma manera que pueden 
  implementar múltiples interfaces en C#.

- No pueden ser subclasificadas.

- Se asignan en la pila (stack) por defecto, a menos que:
  - En .NET, se haga [boxing] o se castee a una interfaz.
  - En Rust, se envuelvan en un puntero inteligente como `Box`, `Rc`/`Arc`.

  [boxing]: https://learn.microsoft.com/es-es/dotnet/csharp/programming-guide/types/boxing-and-unboxing

En C#, un `struct` es una forma de modelar un _[value type]_ ([tipos de valor])
en .NET, que suele ser algún primitivo específico del dominio o compuesto con 
[semántica de igualdad] de valores. En Rust, un `struct` es la construcción 
principal para modelar cualquier estructura de datos (la otra siendo un `enum`).

  [value type]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types
  [tipos de valor]: https://learn.microsoft.com/es-es/dotnet/csharp/language-reference/builtin-types/value-types
  [semántica de igualdad]: https://learn.microsoft.com/es-es/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type

Un `struct` (o `record struct`) en C# tiene copia por valor y semántica de 
igualdad de valores por defecto, pero en Rust, esto requiere simplemente un paso
más utilizando [el atributo `#derive`][derive] y enumerando los traits que se 
deben implementar:

  [derive]: https://doc.rust-lang.org/stable/reference/attributes/derive.html

```rust
#[derive(Clone, Copy, PartialEq, Eq, Hash)]
struct Point {
    x: i32,
    y: i32,
}
```

En C#/.NET, los Value Types suelen ser diseñados por un desarrollador para ser
inmutables. Se considera una práctica recomendada desde el punto de vista 
semántico, pero el lenguaje no impide diseñar un `struct` que realice 
modificaciones destructivas o en el lugar. En Rust, es lo mismo. Un tipo debe 
ser conscientemente desarrollado para ser inmutable.

Dado que Rust no tiene clases y, en consecuencia, jerarquías de tipos basadas en
la subclase, el comportamiento compartido se logra mediante traits y genéricos, 
y el polimorfismo a través de la despacho virtual utilizando [trait objects].

  [trait objects]: https://rustlanges.github.io/rust-book-es/ch17-02-trait-objects.html

Considera la siguiente `struct` que representa un rectángulo en C#:

```c#
struct Rectangle
{
    public Rectangle(int x1, int y1, int x2, int y2) =>
        (X1, Y1, X2, Y2) = (x1, y1, x2, y2);

    public int X1 { get; }
    public int Y1 { get; }
    public int X2 { get; }
    public int Y2 { get; }

    public int Length => Y2 - Y1;
    public int Width => X2 - X1;

    public (int, int) TopLeft => (X1, Y1);
    public (int, int) BottomRight => (X2, Y2);

    public int Area => Length * Width;
    public bool IsSquare => Width == Length;

    public override string ToString() => $"({X1}, {Y1}), ({X2}, {Y2})";
}
```

El equivalente en Rust sería:

```rust
#![allow(dead_code)]

struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    pub fn x1(&self) -> i32 { self.x1 }
    pub fn y1(&self) -> i32 { self.y1 }
    pub fn x2(&self) -> i32 { self.x2 }
    pub fn y2(&self) -> i32 { self.y2 }

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn top_left(&self) -> (i32, i32) {
        (self.x1, self.y1)
    }

    pub fn bottom_right(&self) -> (i32, i32) {
        (self.x2, self.y2)
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }

    pub fn is_square(&self)  -> bool {
        self.width() == self.length()
    }
}

use std::fmt::*;

impl Display for Rectangle {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "({}, {}), ({}, {})", self.x1, self.y2, self.x2, self.y2)
    }
}
```

Ten en cuenta que un `struct` en C# hereda el método `ToString` de `object` y, 
por lo tanto, _anula_ la implementación base para proporcionar una 
representación de cadena personalizada. Dado que no hay herencia en Rust, la 
forma en que un tipo indica el soporte para alguna representación _formateada_ 
es mediante la implementación del trait `Display`. Esto permite que una 
instancia de la estructura participe en el formateo, como se muestra en la 
llamada a `println!` a continuación:

```rust
fn main() {
    let rect = Rectangle::new(12, 34, 56, 78);
    println!("Rectangle = {rect}");
}
```

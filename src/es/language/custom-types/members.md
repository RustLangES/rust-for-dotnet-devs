# Miembros

## Constructores

Rust no tiene ninguna noción de constructores. En su lugar, simplemente escribes
funciones factory que retornan una instancia del tipo. Las funciones Factory 
pueden ser independientes o _funciones asociadas_ al tipo. En términos de C# las
funciones asociadas son como tener metodos estaticos en un tipo. 
Por convención, si hay solo una función factory para una estructura, se le
llama `new`:

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }
}
```

Dado que las funciones en Rust (ya sean asociadas u otras) no admiten sobrecarga;
las funciones factory tienen que tener nombres únicos. Por ejemplo, a 
continuación se presentan algunos ejemplos de las funciones constructores o 
factory disponibles en `String`.

- `String::new`: crea un string vació.
- `String::with_capacity`: crea un string con una capacidad de buffer inicial.
- `String::from_utf8`: crea un string desde bytes de texto codificado en UTF-8.
- `String::from_utf16`: crea un string desde bytes de texto codificado en UTF-16.

En el caso de un tipo `enum` en Rust, las variantes actúan como constructores.
Mira [la sección de tipos Enumerados][ennums] para ver más.

Mira también:

- [Constructors are static, inherent methods (C-CTOR)][rs-api-C-CTOR]

  [enums]: enums.md
  [rs-api-C-CTOR]: https://rust-lang.github.io/api-guidelines/predictability.html?highlight=new#constructors-are-static-inherent-methods-c-ctor

## Métodos (estáticos y basados en instancias)

Al igual que en C#, los tipos de Rust (tanto `enum` como `struct`) pueden tener
métodos estáticos y basados en instancias. En la terminología de Rust, un método
siempre es basado en instancia y se identifica por el hecho de que su primer 
parametro se llama `self`. El parametro `self` no tiene una anotación de tipo,
ya que siempre es el tipo al que pertenece el método. Un método estático se 
llama función asociada. En el ejemplo de a continuación, `new` es una función 
asociada y el resto (`length`, `width`, y `area`) son métodos de el tipo.

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }
}
```

## Constantes

Al igual que en C#, un tipo en Rust puede tener constantes. Sin embargo, el 
aspecto más interesante de notar es que Rust permite que una instancia de tipo
se defina como una constante. 

```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    const ZERO: Point = Point { x: 0, y: 0 };
}
```

En C#, lo mismo requeriría un campo de solo lectura estático.

```c#
readonly record struct Point(int X, int Y)
{
    public static readonly Point Zero = new(0, 0);
}
```

## Eventos

Rust no tiene soporte incorporado para que los miembros de tipo anuncien y 
disparen eventos, como lo tiene C# con la palabra clave `event`.

## Propiedades

En C#, los campos de un tipo suelen ser privados. Luego, se protegen/encapsulan
mediante miembros de propiedades miembro como métodos de acceso (get y set) para 
leer o escribir, para validar el valor al establecerlo o calcular un valor al 
leerlo. Rust solo tiene métodos [donde un getter tiene el mismo nombre que el 
campo (En Rust, los nombres de los métodos pueden compartir el mismo 
identificador que un campo) y el setter utiliza un prefijo `set_`][get-set-name.rs]

  [get-set-name.rs]: https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md#gettersetter-apis

A continuación, se muestra un ejemplo que muestra cómo suelen lucir los métodos
de acceso similares a propiedades para un tipo en Rust:

```rust
struct Rectangle {
    x1: i32, y1: i32,
    x2: i32, y2: i32,
}

impl Rectangle {
    pub fn new(x1: i32, y1: i32, x2: i32, y2: i32) -> Self {
        Self { x1, y1, x2, y2 }
    }

    // como getters de propiedades (cada uno comparte el mismo nombre que el campo)

    pub fn x1(&self) -> i32 { self.x1 }
    pub fn y1(&self) -> i32 { self.y1 }
    pub fn x2(&self) -> i32 { self.x2 }
    pub fn y2(&self) -> i32 { self.y2 }

    // como setters de propiedades

    pub fn set_x1(&mut self, val: i32) { self.x1 = val }
    pub fn set_y1(&mut self, val: i32) { self.y1 = val }
    pub fn set_x2(&mut self, val: i32) { self.x2 = val }
    pub fn set_y2(&mut self, val: i32) { self.y2 = val }

    // como propiedades calculadas

    pub fn length(&self) -> i32 {
        self.y2 - self.y1
    }

    pub fn width(&self)  -> i32 {
        self.x2 - self.x1
    }

    pub fn area(&self)  -> i32 {
        self.length() * self.width()
    }
}
```

## Métodos de Extensión

Los métodos de extensión en C# permiten al desarrollador adjuntar nuevos métodos
vinculados estáticamente a tipos existentes, sin necesidad de modificar la 
definición original del tipo. En el siguiente ejemplo de C#, se añade un nuevo 
método `Wrap` a la clase `StringBuilder` _mediante una extensión_:

```csharp
using System;
using System.Text;
using Extensions; // (1)

var sb = new StringBuilder("Hello, World!");
sb.Wrap(">>> ", " <<<"); // (2)
Console.WriteLine(sb.ToString()); // Muestra: >>> Hello, World! <<<

namespace Extensions
{
    static class StringBuilderExtensions
    {
        public static void Wrap(this StringBuilder sb,
                                string left, string right) =>
            sb.Insert(0, left).Append(right);
    }
}
```

Ten en cuenta que para que un método de extensión esté disponible (2), se debe 
importar el namespace con el tipo que contiene el método de 
extensión (1). Rust ofrece una facilidad muy similar a través de traits, 
llamada _extension traits_. El siguiente ejemplo en Rust es equivalente al 
ejemplo de C# anterior; extiende `String` con el método `wrap`:

```rust
#![allow(dead_code)]

mod exts {
    pub trait StrWrapExt {
        fn wrap(&mut self, left: &str, right: &str);
    }

    impl StrWrapExt for String {
        fn wrap(&mut self, left: &str, right: &str) {
            self.insert_str(0, left);
            self.push_str(right);
        }
    }
}

fn main() {
    use exts::StrWrapExt as _; // (1)

    let mut s = String::from("Hello, World!");
    s.wrap(">>> ", " <<<"); // (2)
    println!("{s}"); // Prints: >>> Hello, World! <<<
}
```

Al igual que en C#, para que el método en el trait de extensión esté 
disponible (2), el trait de extensión debe importarse (1). También ten en cuenta 
que el identificador del trait de extensión `StrWrapExt` puede descartarse 
mediante `_` en el momento de la importación sin afectar la disponibilidad de 
`wrap` para `String`.

## Modificadores de Visibilidad/Acceso

C# tiene varios modificadores de accesibilidad o visibilidad:

- `private`
- `protected`
- `internal`
- `protected internal` (familia)
- `public`

En Rust, una compilación se compone de un árbol de módulos en el que los módulos
contienen y definen [_elementos_][items] como tipos, traits, enums, constantes y
funciones. Casi todo es privado por defecto. Una excepción es, por ejemplo, 
_elementos asociados_ en un trait público, que son públicos por defecto. Esto es
similar a cómo los miembros de una interfaz de C# declarados sin ningún 
modificador público en el código fuente son públicos por defecto. Rust solo 
tiene el modificador `pub` para cambiar la visibilidad con respecto al árbol de 
módulos. Hay variaciones de `pub` que cambian el alcance de la visibilidad 
pública:

- `pub(self)`
- `pub(super)`
- `pub(crate)`
- `pub(in PATH)`

Para obtener más detalles, consulta la sección [Visibility and Privacy][privis] 
en la referencia de Rust.

  [privis]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
  [items]: https://doc.rust-lang.org/reference/items.html

La tabla a continuación es una aproximación de la correspondencia entre los 
modificadores de C# y Rust:

| C#                            | Rust         | Note          |
| ----------------------------- | ------------ | ------------- |
| `private`                     | (default)    | Mirar nota 1. |
| `protected`                   | N/A          | Mirar nota 2. |
| `internal`                    | `pub(crate)` |               |
| `protected internal` (familia)| N/A          | Mirar nota 2. |
| `public`                      | `pub`        |               |

1. No existe una palabra clave para denotar visibilidad privada; es la 
configuración predeterminada en Rust.

2. Dado que no hay jerarquías de tipos basadas en clases en Rust, no hay un 
equivalente de `protected`.

## Mutabilidad

Al diseñar un tipo en C#, es responsabilidad del desarrollador decidir si un 
tipo es mutable o inmutable; si admite mutaciones destructivas o no destructivas. 
C# admite un diseño inmutable para tipos con una _positional record declaration_ 
(`record class` o `readonly record struct`). 
En Rust, la mutabilidad se expresa en los métodos a través del tipo del 
parámetro `self`, como se muestra en el siguiente ejemplo:

```rust
struct Point { x: i32, y: i32 }

impl Point {
    pub fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }

    // self no es mutable

    pub fn x(&self) -> i32 { self.x }
    pub fn y(&self) -> i32 { self.y }

    // self es mutable

    pub fn set_x(&mut self, val: i32) { self.x = val }
    pub fn set_y(&mut self, val: i32) { self.y = val }
}
```

En C#, puedes realizar mutaciones no destructivas usando `with`:

```c#
var pt = new Point(123, 456);
pt = pt with { X = 789 };
Console.WriteLine(pt.ToString()); // Muestra: Point { X = 789, Y = 456 }

readonly record struct Point(int X, int Y);
```

No hay `with` en Rust, pero para emular algo similar en Rust, debe estar 
integrado en el diseño del tipo:

```rust
struct Point { x: i32, y: i32 }

impl Point {
    pub fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }

    pub fn x(&self) -> i32 { self.x }
    pub fn y(&self) -> i32 { self.y }

    // los siguientes métodos consumen self y devuelven una nueva instancia:

    pub fn set_x(self, val: i32) -> Self { Self::new(val, self.y) }
    pub fn set_y(self, val: i32) -> Self { Self::new(self.x, val) }
}
```

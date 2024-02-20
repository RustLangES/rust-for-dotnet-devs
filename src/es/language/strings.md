# Strings

Existen dos tipos de strings en Rust: `String` and `&str`. El primero es
alocado en el monticulo (heap) y el ultimo es un slice de `String` o un `&str`.

> Nota: Slice significa rebana, parte, etc. quiere decir que es una porción de
> un texto.

La comparación de estos a .NET es mostrada en la siguiente tabla:

| Rust               | .NET                 | Nota          |
| ------------------ | -------------------- | --------------|
| `&mut str`         | `Span<char>`         |               |
| `&str`             | `ReadOnlySpan<char>` |               |
| `Box<str>`         | `String`             | mirar Nota 1. |
| `String`           | `String`             |               |
| `String` (mutable) | `StringBuilder`      | mirar Nota 1. |

Hay diferencias en trabajar con strings en Rust y .Net, pero los equivalentes de
arriba deberian de ser un buen punto de inicio. Una de las diferencias es que 
los strings de Rust son codificados en UTF-8, pero los strings de .NET son 
codificados en UTF-16.
Además los strings de .Net son inmutables, pero los strings en Rust pueden ser
mutables cuando se los declara como tal. por ejemplo 
`let s = &mut String::from("hello");`

Hay también diferencias en usar strings debido al concepto del ownership. Para
leer más acerca del ownership con el tipo String, mira [el libro de Rust][ownership-string-type-example].

[ownership-string-type-example]: https://rustlanges.github.io/rust-book-es/ch04-01-what-is-ownership.html#el-tipo-string

Notas

1. El tipo `Box<str>` en Rust es equivalente a el tipo `String` en .NET. La
    diferencia entre los tipos `Box<str>` y `String` en Rust es que el primero 
    almacena el puntero y el tamaño mientras que el segundo almacena puntero, 
    tamaño y capacidad, permitiendo al `String` crecer en tamaño. Este es 
    similar al el tipo `StringBuilder` de .NET cuando el String de Rust es 
    declarado como mutable.

C#:

```csharp
ReadOnlySpan<char> span = "Hello, World!";
string str = "Hello, World!";
StringBuilder sb = new StringBuilder("Hello, World!");
```

Rust:

```rust
let span: &str = "Hello, World!";
let str = Box::new("Hello World!");
let mut sb = String::from("Hello World!");
```

## String Literales

Las literales de cadena en .NET son tipos `String` inmutables y alocados en el 
heap (montículo). En Rust, son `&'static str`, que es inmutable, tiene un 
tiempo de vida global y no se asigna en el montículo; están integradas en el 
binario compilado.

C#

```csharp
string str = "Hello, World!";
```

Rust

```rust
let str: &'static str = "Hello, World!";
```

En C# los strings literales de [verbatim] son equivalentes a los string 
literales sin procesar en Rust.

[verbatim]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/verbatim

C#

```csharp
string str = @"Hello, \World/!";
```

Rust

```rust
let str = r#"Hello, \World/!"#;
```

En C# los string literales UTF-8 en C# son equivalentes a las string literales 
de bytes en Rust.

C#

```csharp
ReadOnlySpan<byte> str = "hello"u8;
```

Rust

```rust
let str = b"hello";
```

## Interpolación de Strings

C# tiene una característica incorporada de interpolación de cadenas que te 
permite incrustar expresiones dentro de una cadena literal. El siguiente 
ejemplo muestra cómo usar la interpolación de cadenas en C#:

```csharp
string name = "John";
int age = 42;
string str = $"Person {{ Name: {name}, Age: {age} }}";
```

Rust no tiene una característica incorporada de interpolación de cadenas. En su 
lugar, se utiliza la macro `format!` para formatear una cadena. El siguiente 
ejemplo muestra cómo usar la interpolación de cadenas en Rust:

```rust
let name = "John";
let age = 42;
let str = format!("Person {{ name: {name}, age: {age} }}");
```

Las clases y structs personalizados también se pueden interpolar en C# debido a 
que el método `ToString()` está disponible para cada tipo al heredar de 
`object`.

```csharp
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public override string ToString() =>
        $"Person {{ Name: {Name}, Age: {Age} }}";
}

var person = new Person { Name = "John", Age = 42 };
Console.Writeline(person);
```

En Rust, no hay un formato predeterminado implementado o heredado para cada 
tipo. En su lugar, se debe implementar el trait `std::fmt::Display` para cada 
tipo que necesite ser convertido a una cadena.

```rust
use std::fmt::*;

struct Person {
    name: String,
    age: i32,
}

impl Display for Person {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        write!(f, "Person {{ name: {}, age: {} }}", self.name, self.age)
    }
}

let person = Person {
    name: "John".to_owned(),
    age: 42,
};

println!("{person}");
```

Otra opción es utilizar el trait `std::fmt::Debug`. El trait `Debug` está 
implementado para todos los tipos estándar y se puede usar para imprimir la 
representación interna de un tipo. El siguiente ejemplo muestra cómo utilizar 
el atributo `derive` para imprimir la representación interna de una estructura 
personalizada utilizando la macro `Debug`. Esta declaración se utiliza para 
implementar automáticamente el trait `Debug` para la estructura `Person`:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: i32,
}

let person = Person {
    name: "John".to_owned(),
    age: 42,
};

println!("{person:?}");
```

> Nota: El uso del especificador de formato `:?` utilizará el trait `Debug` para 
> imprimir la estructura, mientras que omitirlo utilizará el trait `Display`.

Mira también:

- [Rust by Example - Debug](https://doc.rust-lang.org/stable/rust-by-example/hello/print/print_debug.html?highlight=derive#debug)

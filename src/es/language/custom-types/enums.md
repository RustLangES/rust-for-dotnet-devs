# Tipos Enumeración (`enum`)

En C#, un `enum` es un tipo de valor que asigna nombres simbólicos a valores 
enteros:

```c#
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}
```

Rust tiene una sintaxis prácticamente _idéntica_ para hacer lo mismo:

```rust
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}
```

A diferencia de en .NET, una instancia de un tipo `enum` en Rust no tiene ningún
comportamiento predefinido que se herede. Ni siquiera puede participar en 
comprobaciones de igualdad tan simples como `dow == DayOfWeek::Friday`. Para 
hacerlo en cierta medida comparable en función con un `enum` en C#, utiliza 
[el atributo `#derive`][derive] para que los macros implementen automáticamente 
la funcionalidad comúnmente necesaria:

```rust,does_not_compile
#[derive(Debug,     // habilita el formateo en "{:?}"
         Clone,     // requerido por Copy
         Copy,      // habilita la semántica de copia por valor
         Hash,      // habilita la posibilidad de usar en tipos de mapa
         PartialEq  // habilita la igualdad de valores (==)
)]
enum DayOfWeek
{
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
}

fn main() {
    let dow = DayOfWeek::Wednesday;
    println!("Day of week = {dow:?}");

    if dow == DayOfWeek::Friday {
        println!("Yay! It's the weekend!");
    }

    // coerce to integer
    let dow = dow as i32;
    println!("Day of week = {dow:?}");

    let dow = dow as DayOfWeek;
    println!("Day of week = {dow:?}");
}
```

Como muestra el ejemplo anterior, un `enum` puede ser convertido a su valor 
integral asignado, pero lo contrario no es posible como en C# (aunque esto a 
veces tiene la desventaja en C#/.NET de que una instancia de `enum` puede 
contener un valor no representado). En su lugar, depende del desarrollador 
proporcionar una función auxiliar de este tipo:

```rust
impl DayOfWeek {
    fn from_i32(n: i32) -> Result<DayOfWeek, i32> {
        use DayOfWeek::*;
        match n {
            0 => Ok(Sunday),
            1 => Ok(Monday),
            2 => Ok(Tuesday),
            3 => Ok(Wednesday),
            4 => Ok(Thursday),
            5 => Ok(Friday),
            6 => Ok(Saturday),
            _ => Err(n)
        }
    }
}
```

La función `from_i32` devuelve un `DayOfWeek` en un `Result` indicando éxito 
(`Ok`) si `n` es válido. De lo contrario, devuelve `n` tal cual en un `Result` 
que indica fallo (`Err`):

```rust
let dow = DayOfWeek::from_i32(5);
println!("{dow:?}"); // prints: Ok(Friday)

let dow = DayOfWeek::from_i32(50);
println!("{dow:?}"); // prints: Err(50)
```

Existen crates en Rust que pueden ayudar a implementar este mapeo a partir de 
tipos integrales en lugar de tener que codificarlos manualmente.

Un tipo `enum` en Rust también puede servir como una forma de diseñar tipos de 
unión (discriminados), que permiten que diferentes _variantes_ contengan datos 
específicos para cada variante. 
Por ejemplo:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

Esta forma de declaración de `enum` no existe en C#, pero se puede emular con
registros (class records):

```c#
var home = new IpAddr.V4(127, 0, 0, 1);
var loopback = new IpAddr.V6("::1");

abstract record IpAddr
{
    public sealed record V4(byte A, byte B, byte C, byte D): IpAddr;
    public sealed record V6(string Address): IpAddr;
}
```

La diferencia entre ambas es que la definición en Rust produce un 
_tipo cerrado_ sobre las variantes. En otras palabras, el compilador sabe que 
no habrá otras variantes de `IpAddr` excepto `IpAddr::V4` y `IpAddr::V6`, y 
puede utilizar ese conocimiento para realizar verificaciones más estrictas. 
Por ejemplo, en una expresión `match` que es similar a la expresión `switch` en 
C#, el compilador de Rust generará un error a menos que se cubran todas las 
variantes. En cambio, la emulación con C# crea realmente una jerarquía de 
clases (aunque expresada de manera muy concisa) y, dado que `IpAddr` es una 
_clase base abstracta_, el conjunto de todos los tipos que puede representar es 
desconocido para el compilador.

  [derive]: https://doc.rust-lang.org/stable/reference/attributes/derive.html

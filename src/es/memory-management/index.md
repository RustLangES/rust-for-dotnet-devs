# Gestión de Memoria

Al igual que C# y .NET, Rust tiene _memoria segura_ para evitar toda clase de 
errores relacionados con el acceso a la memoria, que terminan siendo la fuente 
de muchas vulnerabilidades de seguridad en el software. Sin embargo, Rust puede 
garantizar la seguridad de memoria en tiempo de compilación; no hay una 
verificación en tiempo de ejecución (como el CLR). La única excepción aquí son 
las verificaciones de límites de arreglos que realiza el código compilado en 
tiempo de ejecución, ya sea el compilador de Rust o el compilador JIT en .NET. 
Al igual que en C#, también es 
[posible escribir código inseguro en Rust][unsafe-rust], y de hecho, ambos 
lenguajes incluso comparten la misma palabra clave, _literalmente_ `unsafe`, 
para marcar funciones y bloques de código donde ya no se garantiza la seguridad 
de memoria.

  [unsafe-rust]: https://book.rustlang-es.org/ch19-01-unsafe-rust

Rust no tiene un garbage collector (GC). Toda la gestión de memoria es 
completamente responsabilidad del desarrollador. Dicho esto, _Rust seguro_ tiene 
reglas rodeando el concepto de _ownership_ que aseguran que la memoria se libere 
_tan pronto como_ ya no esté en uso (por ejemplo, al salir del ámbito de un 
bloque o de una función). El compilador hace un trabajo tremendo, a través del 
análisis estático en tiempo de compilación, para ayudar a gestionar esa memoria 
mediante las reglas de [ownership]. Si se violan, el compilador rechaza el 
código con un error de compilación.

  [ownership]: https://book.rustlang-es.org/ch04-01-what-is-ownership

En .NET, no existe el concepto de ownership de la memoria más allá de las raíces 
del Gargabe Collector (campos estáticos, variables locales en la pila de un hilo, 
registros de la CPU, manejadores, etc.). Es el GC quien recorre desde las raíces 
durante una recolección para determinar toda la memoria en uso siguiendo las 
referencias y purgando el resto. Al diseñar tipos y escribir código, un 
desarrollador de .NET puede permanecer ajeno al ownership, la gestión de memoria
e incluso al funcionamiento del recolector de basura en su mayor parte, excepto 
cuando el código sensible al rendimiento requiere prestar atención a la cantidad 
y la velocidad a la que se asignan objetos en el montón. En contraste, las 
reglas del ownership de Rust requieren que el desarrollador piense y exprese 
explícitamente la propiedad en todo momento y esto impacta todo, desde el diseño 
de funciones, tipos, estructuras de datos hasta la forma en que se escribe el 
código. Además de eso, Rust tiene reglas estrictas sobre cómo se utiliza la 
información, de tal manera que puede identificar en tiempo de compilación 
data [race conditions], así como problemas de corrupción (requiriendo seguridad 
en hilos) que podrían ocurrir potencialmente en tiempo de ejecución. Esta 
sección solo se enfocará en la gestión de memoria y la propiedad.

  [race conditions]: https://doc.rust-lang.org/nomicon/races.html

En Rust, solo puede haber un propietario de una porción de memoria, ya sea en el
stack o en el heap, respaldando una estructura en un momento dado. El compilador 
asigna [lifetimes][lifetimes.rs] y rastrea el ownership. Es posible pasar o 
ceder el ownership, lo cual se denomina _mover_ en Rust. Estas ideas se ilustran 
brevemente en el siguiente código de ejemplo de Rust:

  [lifetimes.rs]: https://doc.rust-lang.org/rust-by-example/scope/lifetime.html

```rust
#![allow(dead_code, unused_variables)]

struct Point {
    x: i32,
    y: i32,
}

fn main() {
  let a = Point { x: 12, y: 34 }; // La instancia de Point es propiedad de a
  let b = a;                      // ahora b es propietario de la instancia de Point
  println!("{}, {}", a.x, a.y);   // ¡error de compilación!
}
```

La primera instrucción en `main` asignará un `Point` y esa memoria será 
propiedad de `a`. En la segunda instrucción, la propiedad se mueve de `a` a `b` 
y `a` ya no puede ser utilizado porque ya no posee nada ni representa una 
memoria válida. La última instrucción que intenta imprimir los campos del punto 
a través de `a` fallará en la compilación. Supongamos que `main` se corrige para 
leerse de la siguiente manera:

```rust
fn main() {
    let a = Point { x: 12, y: 34 }; // La instancia de Point es propiedad de a
    let b = a;                      // ahora b es propietario de la instancia de Point
    println!("{}, {}", b.x, b.y);   // ok, usamos b
}   // Point de b es liberado
```

Nota que cuando `main` termina, `a` y `b` saldrán de su ámbito. La memoria 
detrás de `b` será liberada en virtud de que la pila regresará a su estado 
previo a la llamada de `main`. En Rust, se dice que el punto detrás de `b` fue 
_descartado_. Sin embargo, dado que `a` cedió su propiedad del punto a `b`, no 
hay nada que descartar cuando `a` sale de su ámbito.

Una `struct` en Rust puede definir el código a ejecutar cuando se descarta una 
instancia implementando el trait [`Drop`][drop.rs].

  [drop.rs]: https://doc.rust-lang.org/std/ops/trait.Drop.html

El equivalente aproximado de _dropping_ en C# sería un 
[finalizador de clase][finalizer], pero mientras que un finalizador es llamado 
_automáticamente_ por el GC en algún momento futuro, el _dropping_ en Rust 
siempre es instantáneo y determinista; es decir, ocurre en el punto en que el 
compilador ha determinado que una instancia no tiene propietario basándose en 
los ámbitos y los lifetimes. En .NET, el equivalente de `Drop` sería 
[`IDisposable`][IDisposable] y se implementa en tipos para liberar cualquier 
recurso no administrado o memoria que posean. La _disposición determinística_ no 
está impuesta ni garantizada, pero la declaración `using` en C# se utiliza 
típicamente para delimitar el ámbito de una instancia de un tipo desechable de 
manera que se disponga de manera determinista, al final del bloque de la 
declaración `using`.

  [finalizer]: https://learn.microsoft.com/es-ES/dotnet/csharp/programming-guide/classes-and-structs/finalizers
  [IDisposable]: https://learn.microsoft.com/es-ES/dotnet/api/system.idisposable

Rust tiene la noción de un lifetime global denotada por `'static`, que es un 
especificador de lifetime reservado. Una aproximación muy general en C# serían 
los campos estáticos de solo lectura de los tipos.

En C# y .NET, las referencias se comparten libremente sin mucha consideración, 
por lo que la idea de un único propietario y ceder/mover la propiedad puede 
parecer muy limitante en Rust, pero es posible tener _propiedad compartida_ en 
Rust utilizando el tipo de puntero inteligente [`Rc`][rc.rs]; añade un conteo de 
referencias. Cada vez que [el puntero inteligente es clonado][Rc::clone], se 
incrementa el conteo de referencias. Cuando el clon se descarta, el conteo de 
referencias se decrementa. La instancia real detrás del puntero inteligente se 
descarta cuando el conteo de referencias alcanza cero. Estos puntos se ilustran 
mediante los siguientes ejemplos que se basan en los anteriores:

  [rc.rs]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html
  [Rc::clone]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html#method.clone

```rust
#![allow(dead_code, unused_variables)]

use std::rc::Rc;

struct Point {
    x: i32,
    y: i32,
}

impl Drop for Point {
    fn drop(&mut self) {
        println!("¡Point descartado!");
    }
}

fn main() {
    let a = Rc::new(Point { x: 12, y: 34 });
    let b = Rc::clone(&a); // compartido con b
    println!("a = {}, {}", a.x, a.y); // esta bien usar a
    println!("b = {}, {}", b.x, b.y); // y b
}

// imprime:
// a = 12, 34
// b = 12, 34
// ¡Point descartado!
```

Ten en cuenta que:

- `Point` implementa el método `drop` del trait `Drop` e imprime un mensaje 
  cuando se descarta una instancia de `Point`.

- El punto creado en `main` está envuelto detrás del puntero inteligente `Rc`, 
  por lo que el puntero inteligente _posee_ el punto y no `a`.

- `b` obtiene un clon del puntero inteligente que efectivamente incrementa el 
  conteo de referencias a 2. A diferencia del ejemplo anterior, donde `a` 
  transfirió la propiedad del punto a `b`, tanto `a` como `b` poseen sus propios 
  clones distintos del puntero inteligente, por lo que está bien seguir usando 
  `a` y `b`.

- El compilador habrá determinado que `a` y `b` salen de su ámbito al final de 
  `main` y por lo tanto inyectará llamadas para descartar cada uno. La 
  implementación de `Drop` de `Rc` decrementará el conteo de referencias y 
  también descartará lo que posee si el conteo de referencias ha alcanzado cero. 
  Cuando eso sucede, la implementación de `Drop` de `Point` imprimirá el mensaje, 
  &ldquo;¡Point descartado!&rdquo;. El hecho de que el mensaje se imprima una 
  vez demuestra que solo se creó, compartió y descartó un punto.

`Rc` no es seguro para hilos. Para la propiedad compartida en un programa 
multiproceso, la biblioteca estándar de Rust ofrece [`Arc`][arc.rs] en su lugar. 
El lenguaje Rust evitará el uso de `Rc` entre hilos.

  [arc.rs]: https://doc.rust-lang.org/std/sync/struct.Arc.html

En .NET, los tipos de valor (como `enum` y `struct` en C#) residen en el stack y 
los tipos de referencia (`interface`, `record class` y `class` en C#) se asignan 
en el heap. En Rust, el tipo de tipo (básicamente `enum` o `struct` _en Rust_) 
no determina dónde residirá finalmente la memoria de respaldo. Por defecto, 
siempre está en el stack, pero al igual que .NET y C# tienen la noción de hacer 
boxing de los tipos de valor, lo que los copia al heap, la forma de asignar un 
tipo en el heap es hacer boxing usando [`Box`][box.rs]:

  [box.rs]: https://doc.rust-lang.org/std/boxed/struct.Box.html

```rust
let stack_point = Point { x: 12, y: 34 };
let heap_point = Box::new(Point { x: 12, y: 34 });
```

Al igual que `Rc` y `Arc`, `Box` es un puntero inteligente, pero a diferencia de 
`Rc` y `Arc`, posee exclusivamente la instancia detrás de él. Todos estos 
punteros inteligentes asignan una instancia de su argumento de tipo `T` en 
el heap.

La palabra clave `new` en C# crea una instancia de un tipo, y aunque miembros 
como `Box::new` y `Rc::new` que ves en los ejemplos pueden parecer tener un 
propósito similar, `new` no tiene una designación especial en Rust. Es 
simplemente un _nombre convencional_ que se utiliza para denotar un factory. 
De hecho, se les llama _funciones asociadas_ del tipo, que es la manera de Rust 
de decir métodos estáticos.

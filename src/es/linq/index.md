# LINQ

Esta sección discute LINQ en el contexto y con el propósito de consultar o 
transformar secuencias (`IEnumerable`/`IEnumerable<T>`) y, típicamente, 
colecciones como listas, conjuntos y diccionarios.

## `IEnumerable<T>`

El equivalente de `IEnumerable<T>` en Rust es [`IntoIterator`][into-iter.rs]. 
Así como una implementación de `IEnumerable<T>.GetEnumerator()` devuelve un 
`IEnumerator<T>` en .NET, una implementación de `IntoIterator::into_iter` 
devuelve un [`Iterator`][iter.rs]. Sin embargo, cuando es momento de iterar 
sobre los elementos de un contenedor que anuncia soporte para la iteración a 
través de estos tipos, ambos lenguajes ofrecen azúcar sintáctica en forma de 
constructos de bucles para iterables. En C#, existe `foreach`:

```csharp
using System;
using System.Text;

var values = new[] { 1, 2, 3, 4, 5 };
var output = new StringBuilder();

foreach (var value in values)
{
    if (output.Length > 0)
        output.Append(", ");
    output.Append(value);
}

Console.Write(output); // Imprime: 1, 2, 3, 4, 5
```

En Rust, el equivalente es simplemente `for`:

```rust
use std::fmt::Write;

fn main() {
    let values = [1, 2, 3, 4, 5];
    let mut output = String::new();

    for value in values {
        if output.len() > 0 {
            output.push_str(", ");
        }
        // ! descarta/ignora cualquier error de  write
        _ = write!(output, "{value}");
    }

    println!("{output}");  // Imprime: 1, 2, 3, 4, 5
}
```

El bucle `for` sobre un iterable esencialmente se descompone en lo siguiente:


```rust
use std::fmt::Write;

fn main() {
    let values = [1, 2, 3, 4, 5];
    let mut output = String::new();

    let mut iter = values.into_iter();      // obtiene el iterador
    while let Some(value) = iter.next() {   // en bucle mientras haya más elementos 
        if output.len() > 0 {
            output.push_str(", ");
        }
        _ = write!(output, "{value}");
    }

    println!("{output}");
}
```

Las reglas de ownership y data race conditions de Rust se aplican a todas las 
instancias y datos, y la iteración no es una excepción. Entonces, aunque iterar 
sobre un arreglo pueda parecer sencillo y muy similar a C#, hay que tener en 
cuenta la propiedad cuando se necesita iterar sobre la misma colección/iterable 
más de una vez. El siguiente ejemplo itera la lista de enteros dos veces: una 
vez para imprimir su suma y otra para determinar e imprimir el entero máximo:

```rust
fn main() {
    let values = vec![1, 2, 3, 4, 5];

    // suma todos los valores

    let mut sum = 0;
    for value in values {
        sum += value;
    }
    println!("sum = {sum}");

    // determina el valor maximo

    let mut max = None;
    for value in values {
        if let Some(some_max) = max { // si el máximo está definido
            if value > some_max {     // y el valor es mayor 
                max = Some(value)     // entonces tenemos un nuevo máximo
            }
        } else {                      // el máximo es indefinido cuando la interacción arranca
            max = Some(value)         // entonces establece el primer valor como máximo 
        }
    }
    println!("max = {max:?}");
}
```

Sin embargo, el código anterior es rechazado por el compilador debido a una 
diferencia sutil: `values` ha sido cambiado de un arreglo a un 
[`Vec<int>`][vec.rs], un _vector_, que es el tipo de Rust para arreglos 
dinámicos (similar a `List<T>` en .NET). La primera iteración de `values` 
termina _consumiendo_ cada valor a medida que se suman los enteros. En otras 
palabras, la propiedad de _cada elemento_ en el vector pasa a la variable de 
iteración del bucle: `value`. Dado que `value` sale del alcance al final de cada 
iteración del bucle, la instancia que posee se elimina. Si `values` hubiera sido 
un vector de datos alojados en el heap, la memoria en el heap que respalda cada 
elemento se liberaría a medida que el bucle avanzara al siguiente elemento. Para 
solucionar el problema, uno debe solicitar la iteración sobre 
_referencias compartidas_ usando `&values` en el bucle `for`. Como resultado, 
`value` será una referencia compartida a un elemento en lugar de tomar su 
propiedad.

  [vec.rs]: https://doc.rust-lang.org/stable/std/vec/struct.Vec.html

A continuación se muestra la versión actualizada del ejemplo anterior que 
compila. La corrección consiste simplemente en reemplazar `values` por `&values` 
en cada uno de los bucles `for`.

```rust
fn main() {
    let values = vec![1, 2, 3, 4, 5];

    // suma todos los valores

    let mut sum = 0;
    for value in &values {
        sum += value;
    }
    println!("sum = {sum}");

    // determina el valor máximo

    let mut max = None;
    for value in &values {
        if let Some(some_max) = max { // si el máximo esta definido
            if value > some_max {     // y el valor es mayor
                max = Some(value)     // entonces tenemos un nuevo máximo
            }
        } else {                      // max no esta definido cuando empieza a iterar
            max = Some(value)         // entonces asigna el primer valor
        }
    }
    println!("max = {max:?}");
}
```

El ownership y la liberación de recursos se pueden observar en acción incluso 
cuando `values` es un array en lugar de un vector. Considera solo el bucle de 
suma del ejemplo anterior sobre un array de una estructura que envuelve un 
entero:

```rust
struct Int(i32);

impl Drop for Int {
    fn drop(&mut self) {
        println!("{} liberado", self.0)
    }
}

fn main() {
    let values = [Int(1), Int(2), Int(3), Int(4), Int(5)];
    let mut sum = 0;

    for value in values {
        sum += value.0;
    }

    println!("sum = {sum}");
}
```

`Int` implementa `Drop` para que se imprima un mensaje cuando una instancia se 
libera. Al ejecutar el código anterior, se imprimirá:

    value = Int(1)
    Int(1) liberado
    value = Int(2)
    Int(2) liberado
    value = Int(3)
    Int(3) liberado
    value = Int(4)
    Int(4) liberado
    value = Int(5)
    Int(5) liberado
    sum = 15

Es evidente que cada valor se adquiere y se libera mientras el bucle está en 
ejecución. Una vez que el bucle termina, se imprime la suma. Si `values` en el 
bucle `for` se cambia a `&values`, de esta forma:

```rust
for value in &values {
    // ...
}
```

entonces la salida del programa cambiará radicalmente:

    value = Int(1)
    value = Int(2)
    value = Int(3)
    value = Int(4)
    value = Int(5)
    sum = 15
    Int(1) liberado
    Int(2) liberado
    Int(3) liberado
    Int(4) liberado
    Int(5) liberado

Esta vez, los valores se adquieren pero no se liberan durante el bucle porque 
cada elemento no es poseído por la variable del bucle de iteración. La suma se 
imprime una vez que el bucle termina. Finalmente, cuando el array `values`, que 
aún posee todas las instancias de `Int`, sale de alcance al final de `main`, su 
liberación, a su vez, libera todas las instancias de `Int`.

Estos ejemplos demuestran que, aunque iterar sobre tipos de colecciones puede 
parecer tener muchas similitudes entre Rust y C#, desde las construcciones de 
bucles hasta las abstracciones de iteración, aún existen diferencias sutiles con 
respecto a la propiedad que pueden llevar al compilador a rechazar el código en 
algunos casos.

Mira también:

- [Iterator][iter-mod]
- [Iterando por referencia]

[into-iter.rs]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html
[iter.rs]: https://doc.rust-lang.org/core/iter/trait.Iterator.html
[iter-mod]: https://doc.rust-lang.org/std/iter/index.html
[Iterando por referencia]: https://doc.rust-lang.org/std/iter/index.html#iterating-by-reference

## Operadores

Los _operadores_ en LINQ están implementados en forma de métodos de extensión en 
C# que se pueden encadenar para formar un conjunto de operaciones, siendo lo más 
común la creación de una consulta sobre algún tipo de data source. C# también 
ofrece una _sintaxis de consulta_ inspirada en SQL, con cláusulas como `from`, 
`where`, `select`, `join` y otras, que pueden servir como una alternativa o 
complemento al encadenamiento de métodos. Muchos bucles imperativos pueden 
reescribirse como consultas en LINQ, mucho más expresivas y componibles.

Rust no ofrece nada similar a la sintaxis de consultas de C#. Tiene métodos, 
llamados _[adaptadores]_ en términos de Rust, sobre tipos iterables y, por lo 
tanto, directamente comparables al encadenamiento de métodos en C#. Sin embargo, 
mientras que reescribir un bucle imperativo como código LINQ en C# a menudo es 
beneficioso en términos de expresividad, robustez y componibilidad, existe un 
compromiso con el rendimiento. Los bucles imperativos orientados a cálculos 
_generalmente_ se ejecutan más rápido porque el compilador JIT los puede 
optimizar y se incurren en menos despachos virtuales o invocaciones indirectas 
de funciones. Lo sorprendente en Rust es que no existe tal compromiso de 
rendimiento al elegir usar cadenas de métodos en una abstracción como un 
iterador en lugar de escribir un bucle imperativo manualmente. Por lo tanto, es 
mucho más común ver lo primero en el código.

La siguiente tabla enumera los métodos más comunes de LINQ y sus contrapartes 
aproximadas en Rust:

| .NET              | Rust         | Note         |
| ----------------- | ------------ | ------------ |
| `Aggregate`       | `reduce`     | Mira nota 1. |
| `Aggregate`       | `fold`       | Mira nota 1. |
| `All`             | `all`        |              |
| `Any`             | `any`        |              |
| `Concat`          | `chain`      |              |
| `Count`           | `count`      |              |
| `ElementAt`       | `nth`        |              |
| `GroupBy`         | -            |              |
| `Last`            | `last`       |              |
| `Max`             | `max`        |              |
| `Max`             | `max_by`     |              |
| `MaxBy`           | `max_by_key` |              |
| `Min`             | `min`        |              |
| `Min`             | `min_by`     |              |
| `MinBy`           | `min_by_key` |              |
| `Reverse`         | `rev`        |              |
| `Select`          | `map`        |              |
| `Select`          | `enumerate`  |              |
| `SelectMany`      | `flat_map`   |              |
| `SelectMany`      | `flatten`    |              |
| `SequenceEqual`   | `eq`         |              |
| `Single`          | `find`       |              |
| `SingleOrDefault` | `try_find`   |              |
| `Skip`            | `skip`       |              |
| `SkipWhile`       | `skip_while` |              |
| `Sum`             | `sum`        |              |
| `Take`            | `take`       |              |
| `TakeWhile`       | `take_while` |              |
| `ToArray`         | `collect`    | Mira nota 2. |
| `ToDictionary`    | `collect`    | Mira nota 2. |
| `ToList`          | `collect`    | Mira nota 2. |
| `Where`           | `filter`     |              |
| `Zip`             | `zip`        |              |

1. La sobrecarga de `Aggregate` que no acepta un valor inicial es equivalente a 
    `reduce`, mientras que la sobrecarga de `Aggregate` que acepta un valor 
    inicial corresponde a `fold`.

2. [`collect`][collect.rs] en Rust generalmente funciona para cualquier tipo 
    coleccionable, que se define como [un tipo que puede inicializarse a partir 
    de un iterador (ver `FromIterator`)][FromIter.rs]. `collect` necesita un 
    tipo de destino, que a veces el compilador tiene dificultades para inferir, 
    por lo que el _turbofish_ (`::<>`) se usa a menudo en combinación con él, 
    como en `collect::<Vec<_>>()`. Por esta razón, `collect` aparece junto a 
    varios métodos de extensión de LINQ que convierten una fuente 
    enumerable/iterable en una instancia de algún tipo de colección.

  [FromIter.rs]: https://doc.rust-lang.org/stable/std/iter/trait.FromIterator.html

El siguiente ejemplo muestra lo similar que es transformar secuencias en C# y 
hacer lo mismo en Rust. Primero en C#:

```csharp
var result =
    Enumerable.Range(0, 10)
              .Where(x => x % 2 == 0)
              .SelectMany(x => Enumerable.Range(0, x))
              .Aggregate(0, (acc, x) => acc + x);

Console.WriteLine(result); // 50
```

Y en Rust:

```rust
let result = 
    (0..10)
        .filter(|x| x % 2 == 0)
        .flat_map(|x| (0..x))
        .fold(0, |acc, x| acc + x);

println!("{result}"); // 50
```

[adapters]: https://doc.rust-lang.org/std/iter/index.html#adapters
[collect.rs]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

## Deferred execution (laziness)

Muchos operadores en LINQ están diseñados para ser lazy, de manera que solo 
realizan trabajo cuando es absolutamente necesario. Esto permite la composición 
o encadenamiento de varias operaciones/métodos sin causar efectos secundarios. 
Por ejemplo, un operador LINQ puede devolver un `IEnumerable<T>` que está 
inicializado, pero no produce, calcula ni materializa ningún ítem de `T` hasta 
que se itera sobre él. Se dice que el operador tiene _semántica de ejecución 
diferida_. Si cada `T` se calcula a medida que la iteración llega a él (en lugar 
de cuando comienza la iteración), se dice que el operador _transmite_ los 
resultados.

Los iteradores en Rust tienen el mismo concepto de [_laziness_][iter-laziness] y 
transmisión de resultados.

  [iter-laziness]: https://doc.rust-lang.org/std/iter/index.html#laziness

En ambos casos, esto permite representar _secuencias infinitas_, donde la 
secuencia subyacente es infinita, pero el desarrollador decide cómo debe 
terminarse la secuencia. El siguiente ejemplo muestra esto en C#:

```csharp
foreach (var x in InfiniteRange().Take(5))
    Console.Write($"{x} "); // Muestra "0 1 2 3 4"

IEnumerable<int> InfiniteRange()
{
    for (var i = 0; ; ++i)
        yield return i;
}
```

Rust admite el mismo concepto a través de rangos infinitos:

```rust
// Los generadores y yield en Rust son inestables en este momento, por lo que
// en su lugar, este ejemplo utiliza `Range`:
// https://doc.rust-lang.org/std/ops/struct.Range.html

for value in (0..).take(5) {
    print!("{value} "); // Muestra "0 1 2 3 4"
}
```

## Métodos de Iterador (`yield`)

C# tiene la palabra clave `yield` que permite al desarrollador escribir 
rápidamente un _método de iterador_. El tipo de retorno de un método de iterador 
puede ser un `IEnumerable<T>` o un `IEnumerator<T>`. El compilador convierte el 
cuerpo del método en una implementación concreta del tipo de retorno, en lugar 
de que el desarrollador tenga que escribir una clase completa cada vez. 

_[Coroutines][coroutines.rs]_, como se les llama en Rust, todavía se consideran 
una característica inestable en el momento de escribir esto.

  [coroutines.rs]: https://doc.rust-lang.org/unstable-book/language-features/coroutines.html

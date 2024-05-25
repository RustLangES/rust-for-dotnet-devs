# Sincronización

Cuando los datos son compartidos entre hilos, es necesario sincronizar el acceso 
de lectura y escritura a los datos para evitar la corrupción. En C#, se ofrece 
la palabra clave `lock` como un primitivo de sincronización (que se desenrolla 
en el uso seguro de excepciones de `Monitor` de .NET):

```csharp
using System;
using System.Threading;

var dataLock = new object();
var data = 0;
var threads = new List<Thread>();

for (var i = 0; i < 10; i++)
{
    var thread = new Thread(() =>
    {
        for (var j = 0; j < 1000; j++)
        {
            lock (dataLock)
                data++;
        }
    });
    threads.Add(thread);
    thread.Start();
}

foreach (var thread in threads)
    thread.Join();

Console.WriteLine(data);
```

En Rust, uno debe hacer uso explícito de estructuras de concurrencia como 
`Mutex`:

```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    let data = Arc::new(Mutex::new(0)); // (1)

    let mut threads = vec![];
    for _ in 0..10 {
        let data = Arc::clone(&data); // (2)
        let thread = thread::spawn(move || { // (3)
            for _ in 0..1000 {
                let mut data = data.lock().unwrap();
                *data += 1; // (4)
            }
        });
        threads.push(thread);
    }

    for thread in threads {
        thread.join().unwrap();
    }

    println!("{}", data.lock().unwrap());
}
```

Algunas cosas a tener en cuenta:

- Dado que la propiedad de la instancia de `Mutex` y, a su vez, los datos que 
  protege serán compartidos por múltiples hilos, se envuelve en un `Arc` (1). 
  `Arc` proporciona recuento de referencias atómico, que se incrementa cada vez 
  que se clona (2) y se decrementa cada vez que se elimina. Cuando el recuento 
  alcanza cero, se elimina el mutex y, por lo tanto, los datos que protege. Esto 
  se discute con más detalle en [Gestión de Memoria][Memory Management].

- La closure para cada hilo recibe la propiedad (3) de la 
  _referencia clonada_ (2).

- El código similar a un puntero, que es `*data += 1` (4), no es un acceso a 
  puntero inseguro incluso si parece serlo. Está actualizando los datos 
  _envueltos_ en el [mutex guard].

A diferencia de la versión de C#, donde se puede volver inseguro para hilos al 
comentar la declaración `lock`, la versión de Rust se negará a compilar si se 
cambia de alguna manera (por ejemplo, al comentar partes) que la vuelva insegura 
para hilos. Esto demuestra que escribir código seguro para hilos es 
responsabilidad del desarrollador en C# y .NET mediante el uso cuidadoso de 
estructuras sincronizadas, mientras que en Rust, uno puede confiar en el 
compilador.

El compilador puede ayudar porque las estructuras de datos en Rust están 
marcadas por _traits_ especiales (ver [Interfaces]): `Sync` y `Send`. 
[`Sync`][sync.rs] indica que las referencias a las instancias de un tipo son 
seguras para compartir entre hilos. [`Send`][send.rs] indica que es seguro 
enviar instancias de un tipo a través de los límites de los hilos. Para obtener 
más información, consulta el capítulo 
"[Concurrencia sin miedo][Fearless Concurrency]" del libro de Rust.

  [Fearless Concurrency]: https://book.rustlang-es.org/ch16-00-concurrency
  [Memory Management]: ../memory-management/index.md
  [mutex guard]: https://doc.rust-lang.org/stable/std/sync/struct.MutexGuard.html
  [sync.rs]: https://doc.rust-lang.org/stable/std/marker/trait.Sync.html
  [send.rs]: https://doc.rust-lang.org/stable/std/marker/trait.Send.html
  [interfaces]: ../language/custom-types.md#interfaces

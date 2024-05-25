# Threading

La biblioteca estándar de Rust admite hilos, sincronización y concurrencia. 
Aunque el lenguaje en sí y la biblioteca estándar tienen soporte básico para 
estos conceptos, gran parte de la funcionalidad adicional es proporcionada por 
crates y no se cubrirá en este documento.

A continuación se presenta una lista aproximada de la correspondencia entre los 
tipos y métodos de hilos en .NET y Rust:

| .NET               | Rust                      |
| ------------------ | ------------------------- |
| `Thread`           | `std::thread::thread`     |
| `Thread.Start`     | `std::thread::spawn`      |
| `Thread.Join`      | `std::thread::JoinHandle` |
| `Thread.Sleep`     | `std::thread::sleep`      |
| `ThreadPool`       | -                         |
| `Mutex`            | `std::sync::Mutex`        |
| `Semaphore`        | -                         |
| `Monitor`          | `std::sync::Mutex`        |
| `ReaderWriterLock` | `std::sync::RwLock`       |
| `AutoResetEvent`   | `std::sync::Condvar`      |
| `ManualResetEvent` | `std::sync::Condvar`      |
| `Barrier`          | `std::sync::Barrier`      |
| `CountdownEvent`   | `std::sync::Barrier`      |
| `Interlocked`      | `std::sync::atomic`       |
| `Volatile`         | `std::sync::atomic`       |
| `ThreadLocal`      | `std::thread_local`       |

Lanzar un hilo y esperar a que termine funciona de la misma manera en C#/.NET y 
Rust. A continuación, se muestra un programa simple en C# que crea un hilo 
(donde el hilo imprime algún texto en la salida estándar) y luego espera a que 
termine:

```csharp
using System;
using System.Threading;

var thread = new Thread(() => Console.WriteLine("¡Hola, desde un hilo!"));
thread.Start();
thread.Join(); // espera a que el hilo termine
```

El mismo código en Rust sería el siguiente:

```rust
use std::thread;

fn main() {
    let thread = thread::spawn(|| println!("¡Hola, desde un hilo!"));
    thread.join().unwrap(); // espera a que el hilo termine
}
```

Crear e inicializar un objeto hilo y comenzar un hilo son dos acciones 
diferentes en .NET, mientras que en Rust ambas ocurren al mismo tiempo 
con `thread::spawn`.

En .NET, es posible enviar datos como un argumento a un hilo:

```csharp
#nullable enable

using System;
using System.Text;
using System.Threading;

var t = new Thread(obj =>
{
    var data = (StringBuilder)obj!;
    data.Append(" Mundo!");
});

var data = new StringBuilder("¡Hola");
t.Start(data);
t.Join();

Console.WriteLine($"Frase: {data}");
```

Sin embargo, una versión más moderna o concisa usaría closures:

```csharp
using System;
using System.Text;
using System.Threading;

var data = new StringBuilder("¡Hola");

var t = new Thread(obj => data.Append(" Mundo!"));

t.Start();
t.Join();

Console.WriteLine($"Frase: {data}");
```

En Rust, no hay ninguna variante de `thread::spawn` que haga lo mismo. En su 
lugar, los datos se pasan al hilo mediante un cierre closure:

```rust
use std::thread;

fn main() {
    let data = String::from("¡Hola");
    let handle = thread::spawn(move || {
        let mut data = data;
        data.push_str(" Mundo!");
        data
    });
    println!("Frase: {}", handle.join().unwrap());
}
```

Algunas cosas a tener en cuenta:

- La palabra clave `move` es _necesaria_ para _mover_ o pasar la propiedad de 
  `data` al cierre para el hilo. Una vez hecho esto, ya no es legal seguir 
  utilizando la variable `data` en `main`. Si es necesario, `data` debe ser 
  copiada o clonada (dependiendo de lo que admita el tipo de valor).

- Los hilos de Rust pueden devolver valores, como las tareas en C#, lo que se 
  convierte en el valor de retorno del método `join`.

- Es posible también pasar datos al hilo de C# mediante una closure, como en el 
  ejemplo de Rust, pero la versión de C# no necesita preocuparse por el 
  ownership ya que la memoria detrás de los datos será reclamada por el GC una 
  vez que nadie la esté referenciando.
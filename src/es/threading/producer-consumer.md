# Productor-Consumidor

El patrón productor-consumidor es muy común para distribuir trabajo entre hilos 
donde los datos son pasados desde hilos productores a hilos consumidores sin 
necesidad de compartir o bloquear. .NET tiene un soporte muy amplio para esto, 
pero en el nivel más básico, `System.Collections.Concurrent` proporciona 
`BlockingCollection` como se muestra en el siguiente ejemplo en C#:

```csharp
using System;
using System.Threading;
using System.Collections.Concurrent;

var messages = new BlockingCollection<string>();
var producer = new Thread(() =>
{
    for (var n = 1; i < 10; i++)
        messages.Add($"Mensaje #{n}");
    messages.CompleteAdding();
});

producer.Start();

// el hilo principal es el consumidor aquí
foreach (var message in messages.GetConsumingEnumerable())
    Console.WriteLine(message);

producer.Join();
```

Lo mismo se puede hacer en Rust utilizando _canales_. La biblioteca estándar 
principalmente proporciona `mpsc::channel`, que es un canal que admite múltiples 
productores y un único consumidor. Una traducción aproximada del ejemplo 
anterior en C# a Rust se vería así:

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    let producer = thread::spawn(move || {
        for n in 1..10 {
            tx.send(format!("Mensaje #{}", n)).unwrap();
        }
    });

    // el hilo principal es el consumidor aquí
    for received in rx {
        println!("{}", received);
    }

    producer.join().unwrap();
}
```

Al igual que los canales en Rust, .NET también ofrece canales en el espacio de 
nombres `System.Threading.Channels`, pero está diseñado principalmente para ser 
utilizado con tareas y programación asincrónica mediante el uso de `async` y 
`await`. El equivalente de los [canales amigables para asincronía en el espacio de 
Rust es ofrecido por el runtime de Tokio][tokio-channels].

  [tokio-channels]: https://tokio.rs/tokio/tutorial/channels

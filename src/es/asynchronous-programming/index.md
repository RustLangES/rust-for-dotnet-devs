# Programación Asíncrona

Tanto .NET como Rust admiten modelos de programación asíncronos, los cuales son similares en cuanto a su uso. El siguiente ejemplo muestra, a un nivel muy alto, cómo se ve el código asíncrono en C#:

```csharp
async Task<string> PrintDelayed(string message, CancellationToken cancellationToken)
{
    await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    return $"Message: {message}";
}
```

El código en Rust tiene una estructura similar. El siguiente ejemplo utiliza [async-std] para la implementación de `sleep`:

```rust
use std::time::Duration;
use async_std::task::sleep;

async fn format_delayed(message: &str) -> String {
    sleep(Duration::from_secs(1)).await;
    format!("Message: {}", message)
}
```

1. La palabra clave [`async`][async.rs] en Rust transforma un bloque de código en una máquina de estados que implementa un rasgo llamado [`Future`][future.rs], de manera similar a como el compilador de C# transforma el código `async` en una máquina de estados. En ambos lenguajes, esto permite escribir código asíncrono de manera secuencial.

2. Cabe destacar que, tanto en Rust como en C#, los métodos/funciones asíncronos están precedidos por la palabra clave `async`, pero los tipos de retorno son diferentes. Los métodos asíncronos en C# indican el tipo de retorno completo y real porque puede variar. Por ejemplo, es común ver métodos que devuelven un `Task<T>` mientras que otros devuelven un `ValueTask<T>`. En Rust, basta con especificar el _tipo interno_ `String` porque siempre será _algún futuro_; es decir, un tipo que implementa el rasgo `Future`.

3. Las palabras clave `await` están en posiciones diferentes en C# y Rust. En C#, se espera un `Task` anteponiendo la expresión con `await`. En Rust, al agregar el sufijo `.await` a la expresión se permite _encadenar métodos_, aunque `await` no sea un método.

Ver también:

- [Programación asíncrona en Rust]

[async-std]: https://docs.rs/async-std/latest/async_std/
[async.rs]: https://doc.rust-lang.org/std/keyword.async.html
[future.rs]: https://doc.rust-lang.org/std/future/trait.Future.html
[Programación asíncrona en Rust]: https://rust-lang.github.io/async-book/

## Ejecución de tareas

En el siguiente ejemplo, el método `PrintDelayed` se ejecuta, aunque no se espere su resultado:

```csharp
var cancellationToken = CancellationToken.None;
PrintDelayed("message", cancellationToken); // Imprime "message" después de un segundo.
await Task.Delay(TimeSpan.FromSeconds(2), cancellationToken);

async Task PrintDelayed(string message, CancellationToken cancellationToken)
{
    await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    Console.WriteLine(message);
}
```

En Rust, la misma invocación no imprime nada.

```rust
use async_std::task::sleep;
use std::time::Duration;

#[tokio::main] // se usa para admitir un método principal asíncrono
async fn main() {
    print_delayed("message"); // No imprime nada.
    sleep(Duration::from_secs(2)).await;
}

async fn print_delayed(message: &str) {
    sleep(Duration::from_secs(1)).await;
    println!("{}", message);
}
```

Esto se debe a que los `futures` son "perezosos": no hacen nada hasta que son ejecutados. La forma más común de ejecutar un `Future` es esperarlo con `.await`. Cuando se llama `.await` en un `Future`, intentará ejecutarse hasta completarse. Si el `Future` está bloqueado, cederá el control del hilo actual. Cuando se pueda hacer más progreso, el `Future` será retomado por el ejecutor y continuará su ejecución, permitiendo que `.await` se resuelva (ver [`async/.await`][async-await.rs]).

Mientras que esperar una función funciona dentro de otras funciones `async`, `main` [no puede ser `async`][error-E0752]. Esto se debe a que Rust no proporciona un entorno de ejecución para el código asíncrono. Por lo tanto, existen bibliotecas para ejecutar código asíncrono, llamadas [runtimes asíncronos]. [Tokio][tokio.rs] es uno de estos entornos, y se usa con frecuencia. El atributo [`tokio::main`][tokio-main.rs] en el ejemplo anterior marca la función `async main` como el punto de entrada que será ejecutado por un entorno de ejecución, que se configura automáticamente al usar la macro.

[tokio.rs]: https://crates.io/crates/tokio
[tokio-main.rs]: https://docs.rs/tokio/latest/tokio/attr.main.html
[async-await.rs]: https://rust-lang.github.io/async-book/03_async_await/01_chapter.html#asyncawait
[error-E0752]: https://doc.rust-lang.org/error-index.html#E0752
[runtimes asíncronos]: https://rust-lang.github.io/async-book/08_ecosystem/00_chapter.html#async-runtimes

## Cancelación de tareas

Los ejemplos anteriores en C# incluían pasar un `CancellationToken` a métodos asíncronos, lo que se considera una buena práctica en .NET. Los `CancellationToken`s pueden usarse para abortar una operación asíncrona.

Dado que los `futures` en Rust son inertes (solo progresan cuando son sondeados), la cancelación funciona de manera diferente. Cuando se descarta un `Future`, este no hará más progresos. Además, descartará todos los valores instanciados hasta el punto donde el futuro esté suspendido debido a alguna operación asíncrona pendiente. Por esta razón, la mayoría de las funciones asíncronas en Rust no toman un argumento para indicar cancelación, y es por esto que a veces se refiere a descartar un futuro como _cancelación_.

[`tokio_util::sync::CancellationToken`][cancellation-token.rs] ofrece un equivalente al `CancellationToken` de .NET para señalar y reaccionar ante la cancelación, en los casos en los que implementar el rasgo `Drop` en un `Future` no sea factible.

[cancellation-token.rs]: https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html
[cancellation-token.rs]: https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html

## Ejecución de múltiples tareas

En .NET, se usan frecuentemente `Task.WhenAny` y `Task.WhenAll` para manejar la ejecución de múltiples tareas.

`Task.WhenAny` se completa tan pronto como cualquiera de las tareas lo haga. Tokio, por ejemplo, proporciona la macro [`tokio::select!`][tokio-select] como alternativa a `Task.WhenAny`, lo que significa esperar en múltiples ramas concurrentes.

```csharp
var cancellationToken = CancellationToken.None;

var result =
    await Task.WhenAny(Delay(TimeSpan.FromSeconds(2), cancellationToken),
                       Delay(TimeSpan.FromSeconds(1), cancellationToken));

Console.WriteLine(result.Result); // Esperó 1 segundo.

async Task<string> Delay(TimeSpan delay, CancellationToken cancellationToken)
{
    await Task.Delay(delay, cancellationToken);
    return $"Waited {delay.TotalSeconds} second(s).";
}
```

El mismo ejemplo en Rust:

```rust
use std::time::Duration;
use tokio::{select, time::sleep};

#[tokio::main]
async fn main() {
    let result = select! {
        result = delay(Duration::from_secs(2)) => result,
        result = delay(Duration::from_secs(1)) => result,
    };

    println!("{}", result); // Esperó 1 segundo.
}

async fn delay(delay: Duration) -> String {
    sleep(delay).await;
    format!("Waited {} second(s).", delay.as_secs())
}
```

Nuevamente, hay diferencias cruciales en las semánticas entre los dos ejemplos. Lo más importante es que `tokio::select!` cancelará todas las ramas restantes, mientras que `Task.WhenAny` deja al usuario la responsabilidad de cancelar cualquier tarea en curso.

De manera similar, `Task.WhenAll` puede ser reemplazado con [`tokio::join!`][tokio-join].

[tokio-select]: https://docs.rs/tokio/latest/tokio/macro.select.html
[tokio-join]: https://docs.rs/tokio/latest/tokio/macro.join.html

## Múltiples consumidores

En .NET, una `Task` puede ser usada por múltiples consumidores. Todos ellos pueden esperar la tarea y ser notificados cuando se complete o falle. En Rust, el `Future` no se puede clonar ni copiar, y al usar `await` se transfiere la propiedad. La extensión `futures::FutureExt::shared` crea un manejador clonable de un `Future`, el cual puede distribuirse entre múltiples consumidores.

```rust
use futures::FutureExt;
use std::time::Duration;
use tokio::{select, time::sleep, signal};
use tokio_util::sync::CancellationToken;

#[tokio::main]
async fn main() {
    let token = CancellationToken::new();
    let child_token = token.child_token();

    let bg_operation = background_operation(child_token);

    let bg_operation_done = bg_operation.shared();
    let bg_operation_final = bg_operation_done.clone();

    select! {
        _ = bg_operation_done => {},
        _ = signal::ctrl_c() => {
            token.cancel();
        },
    }

    bg_operation_final.await;
}

async fn background_operation(cancellation_token: CancellationToken) {
    select! {
        _ = sleep(Duration::from_secs(2)) => println!("Operación en segundo plano completada."),
        _ = cancellation_token.cancelled() => println!("Operación en segundo plano cancelada."),
    }
}
```

## Iteración asíncrona

En .NET, existen [`IAsyncEnumerable<T>`][async-enumerable.net] y [`IAsyncEnumerator<T>`][async-enumerator.net], mientras que Rust aún no tiene una API para la iteración asíncrona en la biblioteca estándar. Para soportar la iteración asíncrona, el rasgo [`Stream`][stream.rs] de [`futures`][futures-stream.rs] ofrece un conjunto de funcionalidades comparables.

En C#, escribir iteradores asíncronos tiene una sintaxis comparable a cuando se escriben iteradores sincrónicos:

```csharp
await foreach (int item in RangeAsync(10, 3).WithCancellation(CancellationToken.None))
    Console.Write(item + " "); // Imprime "10 11 12".

async IAsyncEnumerable<int> RangeAsync(int start, int count)
{
    for (int i = 0; i < count; i++)
    {
        await Task.Delay(TimeSpan.FromSeconds(i));
        yield return start + i;
    }
}
```

En Rust, hay varios tipos que implementan el rasgo `Stream`, y por lo tanto pueden usarse para crear flujos, por ejemplo `futures::channel::mpsc`. Para obtener una sintaxis más cercana a la de C#, [`async-stream`][tokio-async-stream] ofrece un conjunto de macros que pueden utilizarse para generar flujos de manera concisa.

```rust
use async_stream::stream;
use futures_core::stream::Stream;
use futures_util::{pin_mut, stream::StreamExt};
use std::{
    io::{stdout, Write},
    time::Duration,
};
use tokio::time::sleep;

#[tokio::main]
async fn main() {
    let stream = range(10, 3);
    pin_mut!(stream); // necesario para la iteración
    while let Some(result) = stream.next().await {
        print!("{} ", result); // Imprime "10 11 12".
        stdout().flush().unwrap();
    }
}

fn range(start: i32, count: i32) -> impl Stream<Item = i32> {
    stream! {
        for i in 0..count {
            sleep(Duration::from_secs(i as _)).await;
            yield start + i;
        }
    }
}
```

[async-enumerable.net]: https://learn.microsoft.com/es-es/dotnet/api/system.collections.generic.iasyncenumerable-1
[async-enumerator.net]: https://learn.microsoft.com/es-es/dotnet/api/system.collections.generic.iasyncenumerator-1
[stream.rs]: https://rust-lang.github.io/async-book/05_streams/01_chapter.html
[futures-stream.rs]: https://docs.rs/futures/latest/futures/stream/trait.Stream.html
[tokio-async-stream]: https://github.com/tokio-rs/async-stream

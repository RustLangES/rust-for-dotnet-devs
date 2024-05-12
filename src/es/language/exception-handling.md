# Manejo de Excepciones

En .Net, una excepción es un tipo que hereda de la clase 
[`System.Exception`][net-system-exception]. Excepción es lanzada si un problema 
ocurre en una sección de código. Un lanzamiento de excepción es pasado hacia arriba 
al stack hasta que la aplicación la maneje o el programa termine.

Rust no tiene excepciones, pero distingue entre errores _recuperables_ y 
_no recuperables_ en su lugar. Un error recuperable representa un problema que
debe ser reportado, pero sin embargo el programa continua. Resultado de 
operaciones que pueden fallar con errores recuperables son de tipo [`Result<T, E>`][rust-result],
en donde `E`es del tipo de variante de error. La macro [`panic!`][panic] detiene
la ejecución cuando el programa encuentra un error irrecuperable. Un error 
irrecuperable es siempre un síntoma de un bug.

## Tipos de errores personalizados

En .Net, una excepción personalizada deriva de la clase `Exception`. La 
documentación en [Cómo crear excepciones definidas por el usuario][net-user-defined-exceptions]
menciona el siguiente ejemplo:

```csharp
public class EmployeeListNotFoundException : Exception
{
    public EmployeeListNotFoundException() { }

    public EmployeeListNotFoundException(string message)
        : base(message) { }

    public EmployeeListNotFoundException(string message, Exception inner)
        : base(message, inner) { }
}
```

En Rust, uno puede implementar el comportamiento básico para los valores erróneos
via implementación de el trait [`Error`][rust-std-error]. La implementación minima
definida por el usuario en Rust:

```rust
#[derive(Debug)]
pub struct EmployeeListNotFound;

impl std::fmt::Display for EmployeeListNotFound {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.write_str("No se pudo encontrar empleado en la lista.")
    }
}

impl std::error::Error for EmployeeListNotFound {}
```

El equivalente para la `Exception.InnerException` de .Net es el método 
`Error::source()` en Rust. Sin embargo, este no requiere proveer una implementación
para `Error::source()`, la implementación por defecto (blanket implementation)
retorna un `None`.

## Elevando excepciones

Para elevar una excepción en C#, lanza una instancia de la excepción:

```csharp
void ThrowIfNegative(int value)
{
    if (value < 0)
    {
        throw new ArgumentOutOfRangeException(nameof(value));
    }
}
```

Para recuperar errores en Rust, retorna una variante de `Ok` o de `Err` desde
un método:

```rust
fn error_if_negative(value: i32) -> Result<(), &'static str> {
    if value < 0 {
        Err("El argumento especificado esta fuera del rango de valores validos. (Parámetro 'value')")
    } else {
        Ok(())
    }
}
```

La macro [`panic!`][panic] crea errores irrecuperables:

```rust
fn panic_if_negative(value: i32) {
    if value < 0 {
        panic!("El argumento especificado esta fuera del rango de valores validos. (Parámetro 'value')")
    }
}
```

## Propagación de error

En .Net, excepciones son pasadas hacia arriba hasta que son tratadas o el programa
termina. En Rust, los errores irrecuperables son similares, pero tratarlos
es poco común.

Los errores recuperables, sin embargo necesitan ser propagados y tratarlos 
explícitamente. Están presentes siempre representados en la firma de funciones o
métodos en Rust. Capturar las excepciones te permiten tomar acciones basadas en 
la presencia o ausencia de errores en C#.

```csharp
void Write()
{
    try
    {
        File.WriteAllText("file.txt", "content");
    }
    catch (IOException)
    {
        Console.WriteLine("Escribiendo el archivo fallo.");
    }
}
```

En Rust, esto es un equivalente aproximado:

```rust
fn write() {
    match std::fs::File::create("temp.txt")
        .and_then(|mut file| std::io::Write::write_all(&mut file, b"content"))
    {
        Ok(_) => {}
        Err(_) => println!("Escribiendo el archivo fallo."),
    };
}
```

Frecuentemente, los errores recuperables necesitan ser propagados en lugar de ser
tratados. Para esto, la firma del metodo necesita ser compatible con el tipo de
error propagado. El [operador `?`][question-mark-operator] propaga errores 
ergonómicamente:

```rust
fn write() -> Result<(), std::io::Error> {
    let mut file = std::fs::File::create("file.txt")?;
    std::io::Write::write_all(&mut file, b"content")?;
    Ok(())
}
```

**Nota**: Para propagar un error con el _question mark operator_ la implementación
del error necesita ser _compatible_, como describimos en [_un atajo para la propagación de errores_][propagating-errors-rust-book]. El tipo más "compatible" es el [trait object] error
`Box<dyn Error>`.

## Stack traces

Lanzar una excepción no capturada en .Net causara que el runtime imprima un 
stack trace que permitirá depurar el problema con contexto adicional.

Para errores irrecuperables en Rust, los [backtraces de `panic!`][panic-backtrace] 
ofrecen un comportamiento similar.

Los errores recuperables en Rust estable no soportan aún los backtraces, pero es
soportado en Rust experimental cuando usamos el [método provide].


[net-system-exception]: https://learn.microsoft.com/es-ES/dotnet/api/system.exception?view=net-6.0
[rust-result]: https://doc.rust-lang.org/std/result/enum.Result.html
[panic-backtrace]: https://www.rustlang-es.org/rust-book-es/ch09-01-unrecoverable-errors-with-panic.html#usando-el-backtrace-de-panic
[net-user-defined-exceptions]: https://learn.microsoft.com/es-ES/dotnet/standard/exceptions/how-to-create-user-defined-exceptions
[rust-std-error]: https://doc.rust-lang.org/std/error/trait.Error.html
[método provide]: https://doc.rust-lang.org/std/error/trait.Error.html#method.provide
[question-mark-operator]: https://doc.rust-lang.org/std/result/index.html#the-question-mark-operator-
[panic]: https://doc.rust-lang.org/std/macro.panic.html
[propagating-errors-rust-book]: https://www.rustlang-es.org/rust-book-es/ch09-02-recoverable-errors-with-result.html#un-atajo-para-propagar-errores-el-operador-
[trait object]: https://doc.rust-lang.org/reference/types/trait-object.html

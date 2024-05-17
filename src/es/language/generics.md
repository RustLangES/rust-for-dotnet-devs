# Genéricos

Los genéricos en C# proporcionan una forma de crear definiciones para tipos y 
métodos que pueden ser parametrizados sobre otros tipos. Esto mejora la 
reutilización de código, la seguridad de tipos y el rendimiento (por ejemplo, 
evita conversiones en tiempo de ejecución). Considera el siguiente ejemplo de un
tipo genérico que agrega una marca de tiempo a cualquier valor:

```csharp
using System;

sealed record Timestamped<T>(DateTime Timestamp, T Value)
{
    public Timestamped(T value) : this(DateTime.UtcNow, value) { }
}
```

Rust también tiene genéricos, como se muestra en el equivalente del ejemplo 
anterior:

```rust
use std::time::*;

struct Timestamped<T> { value: T, timestamp: SystemTime }

impl<T> Timestamped<T> {
    fn new(value: T) -> Self {
        Self { value, timestamp: SystemTime::now() }
    }
}
```

Mira también:

- [Tipos de Datos Genéricos]

[Tipos de Datos Genéricos]: https://book.rustlang-es.org/ch10-01-syntax.html

## Restricciones de tipo genérico

En C#, los [tipos genéricos pueden ser restringidos][type-constraints.cs] usando
la palabra clave where. El siguiente ejemplo muestra tales restricciones en C#:

```csharp
using System;

// Nota: los registros implementan automáticamente `IEquatable`. La siguiente
// implementación muestra esto explícitamente para una comparación con Rust.
sealed record Timestamped<T>(DateTime Timestamp, T Value) :
    IEquatable<Timestamped<T>>
    where T : IEquatable<T>
{
    public Timestamped(T value) : this(DateTime.UtcNow, value) { }

    public bool Equals(Timestamped<T>? other) =>
        other is { } someOther
        && Timestamp == someOther.Timestamp
        && Value.Equals(someOther.Value);

    public override int GetHashCode() => HashCode.Combine(Timestamp, Value);
}
```

Lo mismo se puede lograr en Rust:

```rust
use std::time::*;

struct Timestamped<T> { value: T, timestamp: SystemTime }

impl<T> Timestamped<T> {
    fn new(value: T) -> Self {
        Self { value, timestamp: SystemTime::now() }
    }
}

impl<T> PartialEq for Timestamped<T>
    where T: PartialEq {
    fn eq(&self, other: &Self) -> bool {
        self.value == other.value && self.timestamp == other.timestamp
    }
}
```

En Rust, las restricciones de tipo genérico se llaman [bounds][bounds.rs].

En la versión de C#, las instancias de `Timestamped<T>` solo pueden crearse para
`T` que implementen `IEquatable<T>` ellos mismos, pero ten en cuenta que la 
versión de Rust es más flexible porque `Timestamped<T>` 
_implementa condicionalmente_ `PartialEq`. Esto significa que las instancias de 
`Timestamped<T>` aún pueden crearse para algunos `T` que no son equiparables, 
pero entonces `Timestamped<T>` no implementará la igualdad a través de 
`PartialEq` para dicho `T`.

Mira también:

- [Traits como parametros]
- [Devolviendo tipos que implementan traits]

[type-constraints.cs]: https://learn.microsoft.com/es-es/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters
[bounds.rs]: https://doc.rust-lang.org/rust-by-example/generics/bounds.html
[Traits como parametros]: https://book.rustlang-es.org/ch10-02-traits.html#traits-como-parametros
[Devolviendo tipos que implementan traits]: https://book.rustlang-es.org/ch10-02-traits.html#devolviendo-tipos-que-implementan-traits

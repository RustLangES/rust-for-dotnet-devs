# Administración de Recursos

En la sección anterior, [Administración de Memoria] explicamos la diferencia 
entre .NET y Rust cuando se trata del Garbage Collector, Ownership y finalizadores.
Esto is altamente recomendado para leer.

Esta sección es limitada a proporcionar un ejemplo de una conexión de base de 
datos ficticia que involucra una conexión SQL que debe 
cerrarse/[disposed]/destruirse adecuadamente.

  [disposed]: https://learn.microsoft.com/es-es/dotnet/standard/garbage-collection/implementing-dispose

```csharp
{
    using var db1 = new DatabaseConnection("Server=A;Database=DB1");
    using var db2 = new DatabaseConnection("Server=A;Database=DB2");

    // ...código usando "db1" y "db2"...
}   // "Dispose" de "db1" y "db2" se llamabara aquí; cuando su scope termine

public class DatabaseConnection : IDisposable
{
    readonly string connectionString;
    SqlConnection connection; //Esto implementa IDisposable

    public DatabaseConnection(string connectionString) =>
        this.connectionString = connectionString;

    public void Dispose()
    {
        //Asegurando que se desecha la SqlConnection
        this.connection.Dispose();
        Console.WriteLine("Closing connection: {this.connectionString}");
    }
}
```

```rust
struct DatabaseConnection(&'static str);

impl DatabaseConnection {
    // ...funciones para usar la conexión de base de datos...
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        // ...cerrando la conexión...
        self.close_connection();
        // ...imprimiendo un mensaje...
        println!("Cerrando conexión: {}", self.0)
    }
}

fn main() {
    let _db1 = DatabaseConnection("Server=A;Database=DB1");
    let _db2 = DatabaseConnection("Server=A;Database=DB2");
    // ...code for making use of the database connection...
    // ...codigo para utilizar la conexión a la base de datos...
}   // "Dispose" de "db1" y "db2" se llamabara aquí; cuando su scope termine
```

En .NET, intentar usar un objeto después de llamar a `Dispose` en él típicamente
causará que se lance una `ObjectDisposedException` en tiempo de ejecución. En 
Rust, el compilador garantiza en tiempo de compilación que esto no puede suceder.

[Administración de Memoria]: ../memory-management/index.md

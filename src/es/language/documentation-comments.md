# Comentarios de Documentación

C# provee un mecanismo para documentar las APIs para tipos usando la sintaxis de
un comentario que contiene texto XML. El compilador de C# produce un archivo XML
que contiene estructuras de datos representando el comentario y la firma de la 
API. Otras herramientas pueden procesar la salida para proveer documentación 
legible para humanos en una forma diferente. Un ejemplo simple en C#:

```csharp
/// <summary>
/// Esto es un comentario para documentar <c>MyClass</c>.
/// </summary>
public class MyClass {}
```

En Rust, los [comentarios de documentación] proporcionan el equivalente a los 
comentarios de documentación de C#. Los comentarios de documentación en Rust 
utilizan la sintaxis de Markdown. [rustdoc][rustdoc] es el compilador de 
documentación para el código Rust y generalmente se invoca a través de 
[cargo doc][cargo doc], que compila los comentarios en documentación. 
Por ejemplo:

```rust
/// Este es un comentario de documentación para `MyStruct`.
struct MyStruct;
```

En el .Net SDK hay equivalente a `cargo doc`, como `dotnet doc`.

Mira también:

- [How to write documentation]
- [Documentation tests]

[comentarios de documentación]: https://doc.rust-lang.org/rust-by-example/meta/doc.html
[rustdoc]: https://doc.rust-lang.org/rustdoc/index.html
[cargo doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html
[How to write documentation]: https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html
[documentation tests]: https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html

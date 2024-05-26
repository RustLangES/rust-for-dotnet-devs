# Estructura del Proyecto

Aunque existen convenciones sobre la estructuración de un proyecto en .NET, son 
menos estrictas en comparación con las convenciones de estructura de proyectos 
en Rust. Al crear una solución de dos proyectos usando Visual Studio 2022 (una 
biblioteca de clases y un proyecto de prueba xUnit), se creará la siguiente 
estructura:

```
    .
    |   BibliotecaDeClasesDeEjemplo.sln
    +---BibliotecaDeClasesDeEjemplo
    |       Class1.cs
    |       BibliotecaDeClasesDeEjemplo.csproj
    +---TestDeEjemploDelProjecto
            TestDeEjemploDelProjecto.csproj
            UnitTest1.cs
            Usings.cs
```

- Cada proyecto reside en un directorio separado, con su propio archivo `.csproj`.
- En la raíz del repositorio hay un archivo `.sln`.

Cargo utiliza las siguientes convenciones para la 
[estructura del paquete][package layout] para facilitar la inmersión en un nuevo 
[paquete de Cargo][rust-package]:

```
    .
    +-- Cargo.lock
    +-- Cargo.toml
    +-- src/
    |   +-- lib.rs
    |   +-- main.rs
    +-- benches/
    |   +-- algun-bench.rs
    +-- ejemplos/
    |   +-- algun-ejemplo.rs
    +-- tests/
        +-- algun-test-de-integracion.rs
```

- `Cargo.toml` y `Cargo.lock` se almacenan en la raíz del paquete.
- `src/lib.rs` es el archivo de biblioteca predeterminado, y `src/main.rs` es el 
  archivo ejecutable predeterminado (ver 
  [descubrimiento automático de objetivos][target auto-discovery]).
- Los benchmarks se colocan en el directorio `benches`, las pruebas de 
  integración se colocan en el directorio `tests` (ver 
  [testing][section-testing], [benchmarking][section-benchmarking]).
- Los ejemplos se colocan en el directorio `examples`.
- No hay un crate separado para las pruebas unitarias, las pruebas unitarias 
  viven en el mismo archivo que el código (ver [pruebas][section-testing]).

[package layout]: https://doc.rust-lang.org/cargo/guide/project-layout.html
[rust-package]: https://doc.rust-lang.org/cargo/appendix/glossary.html#package
[target auto-discovery]: https://doc.rust-lang.org/cargo/reference/cargo-targets.html#target-auto-discovery
[section-testing]: ../testing/index.md
[section-benchmarking]: ../benchmarking/index.md

## Gestión de proyectos grandes

Para proyectos muy grandes en Rust, Cargo ofrece 
[workspace][cargo-workspaces] para organizar el proyecto. Un espacio 
de trabajo puede ayudar a gestionar múltiples paquetes relacionados que se 
desarrollan en conjunto. Algunos proyectos utilizan 
[_manifiestos virtuales_][cargo-virtual-manifest], especialmente cuando no hay 
un paquete principal.

[cargo-workspaces]: https://book.rustlang-es.org/ch14-03-cargo-workspaces
[cargo-virtual-manifest]: https://doc.rust-lang.org/cargo/reference/workspaces.html#virtual-workspace

## Gestión de versiones de dependencias

Al gestionar proyectos más grandes en .NET, puede ser apropiado gestionar las 
versiones de las dependencias de forma centralizada, utilizando estrategias como 
la [Gestión Central de Paquetes]. Cargo introdujo la 
[herencia de workspace][workspace inheritance] para gestionar las dependencias de forma 
centralizada.

[Gestión Central de Paquetes]: https://learn.microsoft.com/es-ES/nuget/consume-packages/Central-Package-Management
[workspace inheritance]: https://doc.rust-lang.org/cargo/reference/workspaces.html#the-package-table

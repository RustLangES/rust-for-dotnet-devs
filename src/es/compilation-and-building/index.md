# Compilación y Building

## CLI de .NET

El equivalente de la CLI de .NET (`dotnet`) en Rust es [Cargo] (`cargo`). Ambas 
herramientas son envoltorios de puntos de entrada que simplifican el uso de 
otras herramientas de bajo nivel. Por ejemplo, aunque podrías invocar el 
compilador de C# directamente (`csc`) o MSBuild a través de `dotnet msbuild`, 
los desarrolladores tienden a usar `dotnet build` para construir su solución. De 
manera similar en Rust, aunque podrías usar el compilador de Rust (`rustc`) 
directamente, usar `cargo build` es generalmente mucho más simple.

[cargo]: https://doc.rust-lang.org/cargo/

## Building

Construir un ejecutable en .NET usando [`dotnet build`][net-build-output] 
restaura los paquetes, compila las fuentes del proyecto en un [ensamblado]. El 
ensamblado contiene el código en Lenguaje Intermedio (IL) y _típicamente_ se 
puede ejecutar en cualquier plataforma compatible con .NET, siempre que el 
runtime de .NET esté instalado en el host. Los ensamblados provenientes de 
paquetes dependientes generalmente se ubican junto con el ensamblado de salida 
del proyecto. [`cargo build`][cargo-build] en Rust hace lo mismo, excepto que el 
compilador de Rust enlaza estáticamente (aunque existen otras 
[opciones de enlace][linkage]) todo el código en un solo binario dependiente de 
la plataforma.

Los desarrolladores usan `dotnet publish` para preparar un ejecutable de .NET 
para distribución, ya sea como un _despliegue dependiente del framework_ (FDD) o 
un _despliegue autónomo_ (SCD). En Rust, no hay un equivalente a `dotnet publish` 
ya que la salida de la construcción ya contiene un único binario dependiente de 
la plataforma para cada objetivo.

Al construir una biblioteca en .NET usando [`dotnet build`][net-build-output], 
aún generará un [ensamblado][assembly] que contiene el IL. En Rust, la salida de 
la construcción es, nuevamente, una biblioteca compilada dependiente de la 
plataforma para cada objetivo de biblioteca.

Ver también:

- [Paquetes y Crates][Crate]

[net-build-output]: https://learn.microsoft.com/es-ES/dotnet/core/tools/dotnet-build#description
[assembly]: https://learn.microsoft.com/es-ES/dotnet/standard/assembly/
[cargo-build]: https://doc.rust-lang.org/cargo/commands/cargo-build.html#cargo-build1
[linkage]: https://doc.rust-lang.org/reference/linkage.html
[crate]: https://book.rustlang-es.org/ch07-01-packages-and-crates.html

## Dependencias

En .NET, el contenido de un archivo de proyecto define las opciones de 
compilación y las dependencias. En Rust, al usar Cargo, un archivo `Cargo.toml` 
declara las dependencias de un paquete. Un archivo de proyecto típico se verá 
como:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="morelinq" Version="3.3.2" />
  </ItemGroup>

</Project>
```

El equivalente de `Cargo.toml` en Rust se define como:

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
tokio = "1.0.0"
```

Cargo sigue una convención en la que `src/main.rs` es la raíz del crate binario 
con el mismo nombre que el paquete. Del mismo modo, Cargo sabe que si el 
directorio del paquete contiene `src/lib.rs`, el paquete contiene un crate de 
biblioteca con el mismo nombre que el paquete.

## Paquetes

NuGet se utiliza comúnmente para instalar paquetes, y varias herramientas lo 
soportan. Por ejemplo, añadir una referencia a un paquete NuGet con la CLI de 
.NET añadirá la dependencia al archivo del proyecto:

> dotnet add package morelinq

En Rust, esto funciona de manera casi igual si se usa Cargo para agregar paquetes.

> cargo add tokio

El registro de paquetes más común para .NET es [nuget.org], mientras que los 
paquetes de Rust se comparten generalmente a través de [crates.io].

[nuget.org]: https://www.nuget.org/
[crates.io]: https://crates.io

## Análisis estático de código

Desde .NET 5, los analizadores de Roslyn vienen incluidos con el SDK de .NET y 
proporcionan análisis de calidad de código y estilo de código. La herramienta de 
linting equivalente en Rust es [Clippy].

De manera similar a .NET, donde la compilación falla si hay advertencias al 
configurar [`TreatWarningsAsErrors`][treat-warnings-as-errors] en `true`, Clippy 
puede fallar si el compilador o Clippy emiten advertencias 
(`cargo clippy -- -D warnings`).

Hay otras verificaciones estáticas que considerar agregar a una pipeline de CI 
en Rust:

- Ejecutar [`cargo doc`][cargo-doc] para asegurar que la documentación es 
  correcta.
- Ejecutar [`cargo check --locked`][cargo-check] para asegurar que el archivo 
  `Cargo.lock` está actualizado.

[clippy]: https://github.com/rust-lang/rust-clippy
[treat-warnings-as-errors]: https://learn.microsoft.com/es-ES/dotnet/csharp/language-reference/compiler-options/errors-warnings
[cargo-doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html
[cargo-check]: https://doc.rust-lang.org/cargo/commands/cargo-check.html#manifest-options

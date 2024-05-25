# Compilación Condicional

Tanto .NET como Rust proporcionan la posibilidad de compilar código específico 
basado en condiciones externas.

En .NET es posible utilizar algunas [directivas del preprocesador][preproc-dir] 
para controlar la compilación condicional.

```csharp
#if debug
    Console.WriteLine("Debug");
#else
    Console.WriteLine("No debug");
#endif
```

Además de los símbolos predefinidos, también es posible utilizar la opción del 
compilador _[DefineConstants]_ para definir símbolos que se pueden utilizar con 
`#if`, `#else`, `#elif` y `#endif` para compilar archivos fuente de forma 
condicional en .NET.

En Rust, es posible utilizar el [`atributo cfg`][cfg], el 
[`atributo cfg_attr`][cfg-attr] o el [`macro cfg`][cfg-macro] para controlar la 
compilación condicional.

Al igual que en .NET, además de los símbolos predefinidos, también es posible 
utilizar la [bandera del compilador `--cfg`][cfg-flag] para establecer 
arbitrariamente opciones de configuración.

El [`atributo cfg`][cfg] requiere y evalúa un `ConfigurationPredicate`.

```rust
use std::fmt::{Display, Formatter};

struct MyStruct;

// Esta implementación de Display solo se incluye cuando el SO es Unix pero foo no es igual a bar
// Puedes compilar un ejecutable para esta versión, en Linux, con 'rustc main.rs --cfg foo=\"baz\"'
#[cfg(all(unix, not(foo = "bar")))]
impl Display for MyStruct {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        f.write_str("Ejecutando sin la configuración foo=bar")
    }
}

// Esta función solo se incluye cuando tanto unix como foo=bar están definidos
// Puedes compilar un ejecutable para esta versión, en Linux, con 'rustc main.rs --cfg foo=\"bar\"'
#[cfg(all(unix, foo = "bar"))]
impl Display for MyStruct {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        f.write_str("Ejecutando con la configuración foo=bar")
    }
}

// Esta función provoca un pánico cuando no se compila para Unix
// Puedes compilar un ejecutable para esta versión, en Windows, con 'rustc main.rs'
#[cfg(not(unix))]
impl Display for MyStruct {
    fn fmt(&self, _f: &mut Formatter<'_>) -> std::fmt::Result {
        panic!()
    }
}

fn main() {
    println!("{}", MyStruct);
}
```

El [`atributo cfg_attr`][cfg-attr] incluye condicionalmente atributos basados en 
un predicado de configuración.

```rust
#[cfg_attr(feature = "serialization_support", derive(Serialize, Deserialize))]
pub struct MaybeSerializableStruct;

// Cuando la feature flag `serialization_support` está habilitada, lo anterior se expandirá a:
// #[derive(Serialize, Deserialize)]
// pub struct MaybeSerializableStruct;
```

The built-in [`cfg macro`][cfg-macro] takes in a single configuration predicate
and evaluates to the true literal when the predicate is true and the false
literal when it is false.

El [`macro cfg`][cfg-macro] incorporado toma un solo predicado de configuración 
y evalúa al literal verdadero cuando el predicado es verdadero y al literal 
falso cuando es falso.

```rust
if cfg!(unix) {
  println!("¡Estoy ejecutándome en una máquina Unix!");
}
```

Mira también:

- [Conditional compilation][conditional-compilation]

## Features

La compilación condicional también es útil cuando es necesario proporcionar 
dependencias opcionales. Con las "features" de Cargo, un paquete define 
un conjunto de funcionalidades nombradas en la tabla `[features]` de Cargo.toml, 
y cada funcionalidad puede estar habilitada o deshabilitada. Las funcionalidades 
del paquete que se está construyendo pueden habilitarse en la línea de comandos 
con banderas como `--features`. Las funcionalidades para las dependencias pueden 
habilitarse en la declaración de dependencia en Cargo.toml.

Mira también:

- [Features][features]

[features]: https://doc.rust-lang.org/cargo/reference/features.html
[conditional-compilation]: https://doc.rust-lang.org/reference/conditional-compilation.html#conditional-compilation
[cfg]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute
[cfg-flag]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[cfg-attr]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg_attr-attribute
[cfg-macro]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-macro
[preproc-dir]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives#conditional-compilation
[DefineConstants]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/language#defineconstants

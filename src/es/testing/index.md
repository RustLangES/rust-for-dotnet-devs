# Testing

## Organización de pruebas

Las soluciones .NET utilizan proyectos separados para alojar el código de 
prueba, independientemente del framework de pruebas utilizado (xUnit, NUnit, 
MSTest, etc.) y el tipo de pruebas (unitarias o de integración) que se estén 
escribiendo. Por lo tanto, el código de los test vive en un espacio separado 
al código de la aplicación o biblioteca que se está probando. En Rust, es mucho 
más convencional que las _pruebas unitarias_ se encuentren en un submódulo de 
prueba separado (convencionalmente) llamado `tests`, pero que se coloca en el 
mismo _archivo fuente_ que el código del módulo de aplicación o biblioteca que 
es objeto de las pruebas. Esto tiene dos beneficios:

- El código/módulo y sus pruebas unitarias viven juntos.

- No hay necesidad de un truco como `[InternalsVisibleTo]` que existe en .NET 
  porque las pruebas tienen acceso a los elementos internos al ser un submódulo.

El submódulo de prueba está anotado con el atributo `#[cfg(test)]`, lo que tiene 
el efecto de que todo el módulo se compila (condicionalmente) y se ejecuta solo 
cuando se emite el comando `cargo test`.

Dentro de los submódulos de prueba, las funciones de prueba están anotadas con 
el atributo `#[test]`.

Las pruebas de integración suelen estar en un directorio llamado `tests` que se 
encuentra adyacente al directorio `src` con las pruebas unitarias y el código 
fuente. `cargo test` compila cada archivo en ese directorio como un crate 
separado y ejecuta todos los métodos anotados con el atributo `#[test]`. Dado 
que se entiende que las pruebas de integración están en el directorio `tests`, 
no es necesario marcar los módulos allí con el atributo `#[cfg(test)]`.

Mirar también:

- [Test Organization][test-org]

  [test-org]: https://book.rustlang-es.org/ch11-03-test-organization

## Ejecución de pruebas

Tan simple como puede ser, el equivalente de `dotnet test` en Rust 
es `cargo test`.

El comportamiento predeterminado de `cargo test` es ejecutar todas las pruebas 
en paralelo, pero esto se puede configurar para que se ejecuten de manera 
consecutiva utilizando solo un hilo:

    cargo test -- --test-threads=1

Para obtener más información, consulta 
"[Ejecutando tests en paralelo o consecutivamente][tests-exec]".

  [tests-exec]: https://book.rustlang-es.org/ch11-02-running-tests#
  ejecutando-tests-en-paralelo-o-consecutivamente

## Output en las Pruebas

Para pruebas de integración o de extremo a extremo muy complejas, a veces los 
desarrolladores de .NET registran lo que está sucediendo durante una prueba. La 
forma en que hacen esto varía con cada framework de pruebas. Por ejemplo, en 
NUnit, esto es tan simple como usar `Console.WriteLine`, pero en XUnit, se 
utiliza `ITestOutputHelper`. En Rust, es similar a NUnit; es decir, simplemente 
se escribe en la salida estándar usando `println!`. La salida capturada durante 
la ejecución de las pruebas no se muestra por defecto a menos que `cargo test` 
se ejecute con la opción `--show-output`:

    cargo test --show-output

Para obtener más información, consulta "[Mostrando el Output de las funciones][test-output]".

  [test-output]: https://book.rustlang-es.org/ch11-02-running-tests#
  mostrando-el-output-de-las-funciones

## Aserciones

Los usuarios de .NET tienen múltiples formas de hacer aserciones, dependiendo 
del framework de trabajo que estén utilizando. Por ejemplo, una aserción en 
xUnit.net podría lucir así:

```csharp
[Fact]
public void Something_Is_The_Right_Length()
{
    var value = "something";
    Assert.Equal(9, value.Length);
}
```

Rust no requiere un framework o crate separado. La biblioteca estándar viene con 
_macros_ integradas que son lo suficientemente buenas para la mayoría de las 
afirmaciones en las pruebas:

- [`assert!`][assert]
- [`assert_eq!`][assert_eq]
- [`assert_ne!`][assert_ne]

A continuación se muestra un ejemplo de `assert_eq` en acción:

```rust
#[test]
fn something_is_the_right_length() {
    let value = "something";
    assert_eq!(9, value.len());
}
```

La biblioteca estándar no ofrece nada en la dirección de pruebas basadas en 
datos, como `[Theory]` en xUnit.net.

  [assert]: https://doc.rust-lang.org/std/macro.assert.html
  [assert_eq]: https://doc.rust-lang.org/std/macro.assert_eq.html
  [assert_ne]: https://doc.rust-lang.org/std/macro.assert_ne.html

## Mocking

Cuando se escriben pruebas para una aplicación o biblioteca .NET, existen varios 
framework, como Moq y NSubstitute, para simular las dependencias de los tipos. 
También hay crates similares para Rust, como [`mockall`][mockall], que pueden 
ayudar con la simulación. Sin embargo, también es posible usar 
[compilación condicional][conditional compilation] haciendo uso del 
[atributo `cfg`][cfg-attribute] como un medio simple para la simulación sin 
necesidad de depender de crates o frameworks externos. El atributo `cfg` incluye 
condicionalmente el código que anota en función de un símbolo de configuración, 
como `test` para pruebas. Esto no es muy diferente de usar `DEBUG` para compilar 
condicionalmente código específicamente para compilaciones de depuración. Una 
desventaja de este enfoque es que solo se puede tener una implementación para 
todas las pruebas del módulo.

Cuando se especifica, el atributo `#[cfg(test)]` le indica a Rust que compile y 
ejecute el código solo cuando se ejecute el comando `cargo test`, que ejecuta el 
compilador con `rustc --test`. Lo contrario es cierto para el atributo 
`#[cfg(not(test))]`; incluye el código anotado solo cuando se realiza la prueba 
con `cargo test`.

El ejemplo a continuación muestra la simulación de una función independiente 
`var_os` de la biblioteca estándar que lee y devuelve el valor de una variable 
de entorno. Importa condicionalmente una versión simulada de la función `var_os` 
utilizada por `get_env`. Cuando se compila con `cargo build` o se ejecuta con 
`cargo run`, el binario compilado hará uso de `std::env::var_os`, pero 
`cargo test` en su lugar importará `tests::var_os_mock` como `var_os`, lo que 
hará que `get_env` utilice la versión simulada durante las pruebas:

```rust
// Derechos de autor (c) Microsoft Corporation. Todos los derechos reservados.
// Licenciado bajo la licencia MIT.

/// Función de utilidad para leer una variable de entorno y devolver su valor 
/// si está definida. Falla/pániquea si el valor no es Unicode válido.
pub fn get_env(key: &str) -> Option<String> {
    #[cfg(not(test))]                 // para compilaciones regulares...
    use std::env::var_os;             // ...importar desde la biblioteca estándar
    #[cfg(test)]                      // para compilaciones de prueba...
    use tests::var_os_mock as var_os; // ...importar la simulación desde el submódulo de prueba

    let val = var_os(key);
    val.map(|s| s.to_str()     // obtiene slice de string
                 .unwrap()     // lanza un pánico si no es Unicode válido
                 .to_owned())  // convierte a "String"
}

#[cfg(test)]
mod tests {
    use std::ffi::*;
    use super::*;

    pub(crate) fn var_os_mock(key: &str) -> Option<OsString> {
        match key {
            "FOO" => Some("BAR".into()),
            _ => None
        }
    }

    #[test]
    fn get_env_when_var_undefined_returns_none() {
        assert_eq!(None, get_env("???"));
    }

    #[test]
    fn get_env_when_var_defined_returns_some_value() {
        assert_eq!(Some("BAR".to_owned()), get_env("FOO"));
    }
}
```

  [mockall]: https://docs.rs/mockall/latest/mockall/
  [conditional compilation]: ../conditional-compilation/index.md
  [cfg-attribute]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute


## Cobertura de código

Existe herramientas sofisticadas para .NET en cuanto a análisis de cobertura de 
código de pruebas. En Visual Studio, las herramientas están integradas de forma 
nativa. En Visual Studio Code, existen complementos. Los desarrolladores de .NET 
podrían estar familiarizados con [coverlet] también.

Rust proporciona 
[implementaciones integradas de cobertura de código][built-in-cov] para 
recopilar la cobertura de código de las pruebas.

También hay complementos disponibles para Rust que ayudan con el análisis de 
cobertura de código. No está integrado de manera perfecta, pero con algunos 
pasos manuales, los desarrolladores pueden analizar su código de manera visual.

La combinación del complemento [Coverage Gutters][coverage.gutters] para Visual 
Studio Code y [Tarpaulin] permite el análisis visual de la cobertura de código 
en Visual Studio Code. Coverage Gutters requiere un archivo LCOV. Se pueden usar 
otras herramientas además de [Tarpaulin] para generar ese archivo.

Una vez configurado, ejecuta el siguiente comando:

```bash
cargo tarpaulin --ignore-tests --out Lcov
```

Esto genera un archivo de cobertura de código LCOV. Una vez habilitado 
`Coverage Gutters: Watch`, será recogido por el complemento Coverage Gutters, 
que mostrará indicadores visuales en línea sobre la cobertura de líneas en el 
editor de código fuente.

> Nota: La ubicación del archivo LCOV es esencial. Si hay un espacio de trabajo 
> (ver [Estructura del Proyecto]) con múltiples paquetes y se genera un archivo 
> LCOV en la raíz usando `--workspace`, ese es el archivo que se está 
> utilizando, incluso si hay un archivo presente directamente en la raíz del 
> paquete. Es más rápido aislar el paquete específico bajo prueba en lugar de 
> generar el archivo LCOV en la raíz.

[coverage.gutters]: https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters
[tarpaulin]: https://github.com/xd009642/tarpaulin
[coverlet]: https://github.com/coverlet-coverage/coverlet
[built-in-cov]: https://doc.rust-lang.org/stable/rustc/instrument-coverage.html#test-coverage
[Estructura del Proyecto]: ../project-structure/index.md

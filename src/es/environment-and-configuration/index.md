# Entorno y Configuración

## Accediendo a variables de entorno

.NET proporciona acceso a las variables de entorno a través del método 
`System.Environment.GetEnvironmentVariable`. Este método recupera el valor de 
una variable de entorno en tiempo de ejecución.

```csharp
using System;

const string name = "VARIABLE_EJEMPLO";

var value = Environment.GetEnvironmentVariable(name);
if (string.IsNullOrEmpty(value))
    Console.WriteLine($"Variable '{name}' no esta configurada.");
else
    Console.WriteLine($"Variable '{name}' configurada con '{value}'.");
```

Rust proporciona la misma funcionalidad de acceso a una variable de entorno en 
tiempo de ejecución mediante las funciones `var` y `var_os` del módulo `std::env`.

La función `var` devuelve un `Result<String, VarError>`, devolviendo la variable
si está configurada o devolviendo un error si la variable no está configurada o 
no es Unicode válido.

`var_os` tiene una firma diferente, devolviendo una `Option<OsString>`, 
devolviendo algún valor si la variable está configurada o devolviendo None si la 
variable no está configurada. Un `OsString` no tiene que ser Unicode válido.

```rust
use std::env;


fn main() {
    let key = "VariableEjemplo";
    match env::var(key) {
        Ok(val) => println!("{key}: {val:?}"),
        Err(e) => println!("No se pudo interpretar {key}: {e}"),
    }
}
```

```rust
use std::env;

fn main() {
    let key = "VariableEjemplo";
    match env::var_os(key) {
        Some(val) => println!("{key}: {val:?}"),
        None => println!("{key} no definida en el entorno"),
    }
}
```

Rust también proporciona la funcionalidad de acceder a una variable de entorno 
en tiempo de compilación. El macro `env!` del módulo `std::env` expande el valor 
de la variable en tiempo de compilación, devolviendo un `&'static str`. Si la 
variable no está establecida, se emite un error.

```rust
use std::env;

fn main() {
    let example = env!("VariableEjemplo");
    println!("{example}");
}
```

En .NET, el acceso a variables de entorno en tiempo de compilación se puede 
lograr, de una manera menos directa, a través de 
[generadores de código fuente][source-gen].

[source-gen]: https://learn.microsoft.com/es-ES/dotnet/csharp/roslyn-sdk/source-generators-overview

## Configuración

La configuración en .NET es posible mediante proveedores de configuración. El 
framework proporciona varias implementaciones de proveedores a través del 
espacio de nombres `Microsoft.Extensions.Configuration` y paquetes NuGet.

Los proveedores de configuración leen datos de configuración a partir de pares 
clave-valor utilizando diferentes fuentes y proporcionan una vista unificada de 
la configuración a través del tipo `IConfiguration`.

```csharp
using Microsoft.Extensions.Configuration;

class Example {
    static void Main()
    {
        IConfiguration configuration = new ConfigurationBuilder()
            .AddEnvironmentVariables()
            .Build();

        var example = configuration.GetValue<string>("VariableEjemplo");

        Console.WriteLine(example);
    }
}
```

Otros ejemplos de proveedores se pueden encontrar en la documentación oficial 
[Proveedores de configuración en .NET][conf-net].

Una experiencia de configuración similar en Rust está disponible mediante el uso 
de crates de terceros como [figment] o [config].

Vea el siguiente ejemplo utilizando el crate [config]:

```rust
use config::{Config, Environment};

fn main() {
    let builder = Config::builder().add_source(Environment::default());

    match builder.build() {
        Ok(config) => {
            match config.get_string("variable_ejemplo") {
                Ok(v) => println!("{v}"),
                Err(e) => println!("{e}")
            }
        },
        Err(_) => {
            // algo salio mal
        }
    }
}

```

[conf-net]: https://learn.microsoft.com/es-ES/dotnet/core/extensions/configuration-providers
[figment]: https://crates.io/crates/figment
[config]: https://crates.io/crates/config

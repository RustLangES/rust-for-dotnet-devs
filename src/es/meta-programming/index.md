# Meta Programación

La metaprogramación puede verse como una forma de escribir código que genera o escribe otro código.

Roslyn proporciona una funcionalidad para la metaprogramación en C#, disponible desde .NET 5, llamada [`Generadores de Código`][source-gen]. Los generadores de código pueden crear nuevos archivos fuente de C# en tiempo de compilación, que se agregan a la compilación del usuario. Antes de que se introdujeran los `Generadores de Código`, Visual Studio proporcionaba una herramienta de generación de código a través de las [`Plantillas de Texto T4`][T4]. Un ejemplo de cómo funciona T4 es la siguiente [plantilla] o su [concretización].

Rust también proporciona una funcionalidad para la metaprogramación: [macros]. Existen `macros declarativas` y `macros procedimentales`.

Las macros declarativas permiten escribir estructuras de control que toman una expresión, comparan el valor resultante de la expresión con patrones, y luego ejecutan el código asociado con el patrón coincidente.

El siguiente ejemplo es la definición de la macro `println!`, que es posible llamar para imprimir un texto `println!("Algún texto")`.

```rust
macro_rules! println {
    () => {
        $crate::print!("\n")
    };
    ($($arg:tt)*) => {{
        $crate::io::_print($crate::format_args_nl!($($arg)*));
    }};
}
```

Para aprender más sobre la escritura de macros declarativas, consulta el capítulo de la referencia de Rust sobre [macros por ejemplo] o [El pequeño libro de macros de Rust].

Las [macros procedimentales] son diferentes de las macros declarativas. Estas aceptan un código como entrada, operan sobre ese código y producen un código como salida.

Otra técnica usada en C# para la metaprogramación es la reflexión. Rust no soporta reflexión.

[source-gen]: https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview

## Macros con forma de función

Las macros con forma de función tienen la siguiente forma: `function!(...)`

El siguiente fragmento de código define una macro con forma de función llamada `print_something`, que genera un método `print_it` para imprimir la cadena "Something".

En el archivo lib.rs:

```rust
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn print_something(_item: TokenStream) -> TokenStream {
    "fn print_it() { println!(\"Something\") }".parse().unwrap()
}
```

En el archivo main.rs:

```rust
use replace_crate_name_here::print_something;
print_something!();

fn main() {
    print_it();
}
```

## Macros de tipo derive

Las macros de tipo derive pueden crear nuevos elementos dados el flujo de tokens de una estructura, enumeración o unión. Un ejemplo de una macro derive es `#[derive(Clone)]`, que genera el código necesario para que la estructura/enumeración/unión de entrada implemente el rasgo `Clone`.

Para entender cómo definir una macro derive personalizada, es posible leer la referencia de Rust sobre [macros derive].

[macros derive]: https://doc.rust-lang.org/reference/procedural-macros.html#derive-macros

## Macros de atributo

Las macros de atributo definen nuevos atributos que pueden ser adjuntados a elementos de Rust. Al trabajar con código asincrónico, si se utiliza Tokio, el primer paso será decorar el nuevo `main` asincrónico con una macro de atributo como el siguiente ejemplo:

```rust
#[tokio::main]
async fn main() {
    println!("Hola mundo");
}
```

Para entender cómo definir una macro de atributo personalizada, es posible leer la referencia de Rust sobre [macros de atributo].

[macros de atributo]: https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros

[T4]: https://learn.microsoft.com/es-es/previous-versions/visualstudio/visual-studio-2015/modeling/code-generation-and-t4-text-templates?view=vs-2015&redirectedfrom=MSDN  
[plantilla]: https://github.com/atifaziz/Jacob/blob/master/src/JsonReader.g.tt  
[concretización]: https://github.com/atifaziz/Jacob/blob/master/src/JsonReader.g.cs  
[macros]: https://doc.rust-lang.org/book/ch19-06-macros.html  
[macros por ejemplo]: https://doc.rust-lang.org/reference/macros-by-example.html  
[macros procedimentales]: https://doc.rust-lang.org/reference/procedural-macros.html  
[El pequeño libro de macros de Rust]: https://veykril.github.io/tlborm/

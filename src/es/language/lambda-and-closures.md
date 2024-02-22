# Lambda y Closures

C# y Rust permiten que las funciones sean utilizadas como valores de primera 
clase que posibilitan la escritura de _funciones de orden superior_. Las 
funciones de orden superior son esencialmente funciones que aceptan otras 
funciones como argumentos para permitir que el llamador participe en el código 
de la función llamada. En C#, los _function pointers seguros_ se representan 
mediante delegados, siendo los más comunes `Func` y `Action`. El lenguaje C# 
permite la creación de instancias ad hoc de estos delegados a través de 
_expresiones lambda_.

Rust también tiene function pointers, siendo el tipo más simple `fn`:

```rust
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(|x| x + 1, 5);
    println!("The answer is: {}", answer); // Prints: The answer is: 12
}
```

Sin embargo, Rust hace una distinción entre _funciones punteros_ (donde `fn` 
define un tipo) y _closures_: una closure puede hacer referencia a variables 
desde su ámbito léxico circundante, pero no una función puntero. Aunque C# 
también tiene [function pointers][*delegate] (`*delegate`), el equivalente 
gestionado y seguro para el tipo sería una expresión lambda estática.


  [*delegate]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-9.0/function-pointers

Las funciones y métodos que aceptan closures se escriben con tipos genéricos que
está vinculado a uno de los traits que representa funciones: `Fn`, `FnMut` y 
`FnOnce`. Cuando es el momento de proporcionar un valor para un puntero a 
función o un cierre, un desarrollador de Rust utiliza una _closure expression_ 
(como `|x| x + 1` en el ejemplo anterior), que se traduce de la misma manera que
una expresión lambda en C#. Si la closure expression crea una pointer function  
o una closure depende de si la closure expression hace referencia a su contexto 
o no.

Cuando una closure captura variables de su entorno, entran en juego las reglas 
de ownership porque el ownership termina en la closure. Para obtener más 
información, consulta la sección 
"[Moviendo valores capturados fuera de los closures y los traits Fn][closure-move]" de 
"El Lenguaje de Programación Rust".

  [closure-move]: https://rustlanges.github.io/rust-book-es/ch13-01-closures.html?highlight=moviend#moviendo-valores-capturados-fuera-de-los-closures-y-los-traits-fn

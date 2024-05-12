# Conversión y Casting

Ambos en C# y Rust son estéticamente tipados en tiempo de compilación. Por eso,
luego que una variable es declarada, asignar un valor de un tipo diferente (A
menos que este sea implícitamente convertible a el tipo esperado) de la 
variable esta prohibido. Hay multiples formas para convertir tipos en C# que 
equivalen a Rust.

## Conversiones implícitas

Las conversiones implícitas existen en C# como en Rust (llamadas [cohesiones de tipos]).
Considera el siguiente ejemplo:

```csharp
int intNumber = 1;
long longNumber = intNumber;
```

Rust es mucho más restrictivo al respecto de cual cohesion de tipo permitir:

```rust
let int_number: i32 = 1;
let long_number: i64 = int_number; // error: expected `i64`, found `i32`
```

Un ejemplo para una conversión implícita valida usando [subtipificación][subtyping.rs]
es:

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

Mirar también:

- [cohesion Deref]
- [Subtipificación y varianza]

[cohesiones de tipos]: https://doc.rust-lang.org/reference/type-coercions.html
[subtyping.rs]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md#subtyping
[cohesion Deref]: https://doc.rust-lang.org/std/ops/trait.Deref.html#more-on-deref-coercion
[Subtipificación y varianza]: https://doc.rust-lang.org/reference/subtyping.html#subtyping-and-variance

## Conversión Explicita

Si convertir puede causar perdida de información, C# require conversión explicita
usando una expresión de casting:

```csharp
double a = 1.2;
int b = (int)a;
```

Conversiones explicitas pueden potencialmente fallar durante tiempo de ejecución
con excepciones como `OverflowException` o `InvalidCastException` cuando
se hace _down-casting_.

Rust no provee cohesión entre tipos primitivos, pero en su lugar usa 
[conversion explicita][casting.rs] usando la palabra clave [`as`][as.rs] 
(casting).
Usar Casting en Rust no causara [pánico].

```rust
let int_number: i32 = 1;
let long_number: i64 = int_number as _;
```

[casting.rs]: https://doc.rust-lang.org/rust-by-example/types/cast.html
[as.rs]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#type-cast-expressions

## Conversión Personalizada

Comúnmente, los tipos de .Net proveen operadores de conversión definidos por el 
usuario para convertir un tipo a otro tipo. También, `System.IConvertible` tiene
el propósito de convertir un tipo en otro.

En Rust, la librería estándar contiene una abstracción para convertir un valor en
un tipo diferente, con el trait [`From`][from.rs] y recíprocamente [`Into`][into.rs].
Cuando implementas `From` para un tipo, una implementación por default de `Into`
es automáticamente provista (A esto se le llama _blanket implementation_ en Rust).
El siguiente ejemplo ilustra dos conversiones de tipos.

```rust
fn main() {
    let my_id = MyId("id".into()); // `into()` es implementado automáticamente debido a la implementación del `From<&str>` trait para `String`.
    println!("{}", String::from(my_id)); // Esto usa la implementación `From<MyId>` de `String`.
}

struct MyId(String);

impl From<MyId> for String {
    fn from(MyId(value): MyId) -> Self {
        value
    }
}
```

Mirar también:

- [`TryFrom`][try-from.rs] y [`TryInto`][try-into.rs] para versiones de `From`
  y `Into` que puede fallar.

[from.rs]: https://doc.rust-lang.org/std/convert/trait.From.html
[into.rs]: https://doc.rust-lang.org/std/convert/trait.Into.html
[try-from.rs]: https://doc.rust-lang.org/std/convert/trait.TryFrom.html
[try-into.rs]: https://doc.rust-lang.org/std/convert/trait.TryInto.html
[pánico]: https://www.rustlang-es.org/rust-book-es/ch09-01-unrecoverable-errors-with-panic.html#errores-irrecuperables-con-panic
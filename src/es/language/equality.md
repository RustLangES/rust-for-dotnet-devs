# Equivalencia

Cuando se compara por igualdad en C#, esto se refiere a probar la _equivalencia_ en algunos casos (también conocida como _igualdad de valor_), y en otros casos se refiere a probar la _igualdad de referencia_, que verifica si dos variables se refieren al mismo objeto subyacente en memoria. Cada tipo personalizado puede ser comparado por igualdad porque hereda de `System.Object` (o `System.ValueType` para tipos de valor, que a su vez hereda de `System.Object`), utilizando cualquiera de las semánticas mencionadas anteriormente.

Por ejemplo, al comparar por equivalencia e igualdad de referencia en C#:

```csharp
var a = new Point(1, 2);
var b = new Point(1, 2);
var c = a;
Console.WriteLine(a == b); // (1) True
Console.WriteLine(a.Equals(b)); // (1) True
Console.WriteLine(a.Equals(new Point(2, 2))); // (1) True
Console.WriteLine(ReferenceEquals(a, b)); // (2) False
Console.WriteLine(ReferenceEquals(a, c)); // (2) True

record Point(int X, int Y);
```

1. El operador de igualdad `==` y el método `Equals` en el `record Point` comparan por igualdad de valor, ya que los registros admiten la igualdad de tipo valor de forma predeterminada.

2. Comparar por igualdad de referencia verifica si las variables se refieren al mismo objeto subyacente en memoria.

Equivalente en Rust:

```rust
#[derive(Copy, Clone)]
struct Point(i32, i32);

fn main() {
    let a = Point(1, 2);
    let b = Point(1, 2);
    let c = a;
    println!("{}", a == b); // Error: "an implementation of `PartialEq<_>` might be missing for `Point`"
    println!("{}", a.eq(&b));
    println!("{}", a.eq(&Point(2, 2)));
}
```

El error del compilador anterior ilustra que en Rust las comparaciones de igualdad _siempre_ están relacionadas con una implementación de trait. Para admitir una comparación usando `==`, un tipo debe implementar [`PartialEq`][partialeq.rs].

Corregir el ejemplo anterior significa derivar `PartialEq` para `Point`. Por defecto, al derivar `PartialEq` se compararán todos los campos para la igualdad, por lo que ellos mismos deben implementar `PartialEq`. Esto es comparable a la igualdad de registros en C#.

```rust
#[derive(Copy, Clone, PartialEq)]
struct Point(i32, i32);

fn main() {
    let a = Point(1, 2);
    let b = Point(1, 2);
    let c = a;
    println!("{}", a == b); // true
    println!("{}", a.eq(&b)); // true
    println!("{}", a.eq(&Point(2, 2))); // false
    println!("{}", a.eq(&c)); // true
}
```

Véase también:

- [`Eq`][eq.rs] para una versión más estricta de `PartialEq`

[partialeq.rs]: https://doc.rust-lang.org/std/cmp/trait.PartialEq.html
[eq.rs]: https://doc.rust-lang.org/std/cmp/trait.Eq.html

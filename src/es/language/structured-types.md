# Tipos Estructurados

Tipos de objetos y colecciones comúnmente utilizados en .NET y su mapeo a Rust

| C#           | Rust      |
| ------------ | --------- |
| `Array`      | `Array`   |
| `List`       | `Vec`     |
| `Tuple`      | `Tuple`   |
| `Dictionary` | `HashMap` |

## Array

Los arrays fijos son compatibles de la misma manera en Rust que en .NET.

C#:

```csharp
int[] someArray = new int[2] { 1, 2 };
```

Rust:

```rust
let someArray: [i32; 2] = [1,2];
```

## Listas

En Rust, el equivalente de un `List<T>` es un `Vec<T>`. Los arrays pueden 
convertirse a Vecs y viceversa.

C#:

```csharp
var something = new List<string>
{
    "a",
    "b"
};

something.Add("c");
```

Rust:

```rust
let mut something = vec![
    "a".to_owned(),
    "b".to_owned()
];

something.push("c".to_owned());
```

## Tuplas

C#:

```csharp
var something = (1, 2)
Console.WriteLine($"a = {something.Item1} b = {something.Item2}");
```

Rust:

```rust
let something = (1, 2);
println!("a = {} b = {}", something.0, something.1);

// soporta deconstrucción
let (a, b) = something;
println!("a = {} b = {}", a, b);
```

> **NOTA**: En Rust, los elementos de las tuplas no pueden tener nombres como 
> en C#. La única forma de acceder a un elemento de la tupla es utilizando el 
> índice del elemento o desestructurando la tupla.

## Diccionarios

En Rust el equivalente de un `Dictionary<TKey, TValue>` es un `Hashmap<K, V>`.


C#:

```csharp
var something = new Dictionary<string, string>
{
    { "Foo", "Bar" },
    { "Baz", "Qux" }
};

something.Add("hi", "there");
```

Rust:

```rust
let mut something = HashMap::from([
    ("Foo".to_owned(), "Bar".to_owned()),
    ("Baz".to_owned(), "Qux".to_owned())
]);

something.insert("hi".to_owned(), "there".to_owned());
```

Mirar también:

- [Biblioteca estándar de Rust - Colecciones](https://doc.rust-lang.org/std/collections/index.html)

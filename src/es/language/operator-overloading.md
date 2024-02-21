# Sobrecarga de operadores

Un tipo personalizado puede sobrecargar un _operador sobrecargable_ en C#. 
Considera el siguiente ejemplo en C#:

```csharp
Console.WriteLine(new Fraction(5, 4) + new Fraction(1, 2));  // 14/8

public readonly record struct Fraction(int Numerator, int Denominator)
{
    public static Fraction operator +(Fraction a, Fraction b) =>
        new(a.Numerator * b.Denominator + b.Numerator * a.Denominator, a.Denominator * b.Denominator);

    public override string ToString() => $"{Numerator}/{Denominator}";
}
```

En Rust, muchos operadores [pueden sobrecargarse mediante traits][ops.rs]. Esto 
es posible porque los operadores son azúcar sintáctica para llamadas a métodos. 
Por ejemplo, el operador `+` en `a + b` llama al método `add` (ver 
[sobrecarga de operadores]):

```rust
use std::{fmt::{Display, Formatter, Result}, ops::Add};

struct Fraction {
    numerator: i32,
    denominator: i32,
}

impl Display for Fraction {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        f.write_fmt(format_args!("{}/{}", self.numerator, self.denominator))
    }
}

impl Add<Fraction> for Fraction {
    type Output = Fraction;

    fn add(self, rhs: Fraction) -> Fraction {
        Fraction {
            numerator: self.numerator * rhs.denominator + rhs.numerator * self.denominator,
            denominator: self.denominator * rhs.denominator,
        }
    }
}

fn main() {
    println!(
        "{}",
        Fraction { numerator: 5, denominator: 4 } + Fraction { numerator: 1, denominator: 2 }
    ); // 14/8
}

```

[ops.rs]: https://doc.rust-lang.org/core/ops/
[sobrecarga de operadores]: https://doc.rust-lang.org/rust-by-example/trait/ops.html

# Benchmarking

La ejecución de benchmarks en Rust se realiza a través de 
[`cargo bench`][cargo-bench], un comando específico para `cargo` que ejecuta 
todos los métodos anotados con el atributo `#[bench]`. Este atributo está 
actualmente [inestable][bench-unstable] y disponible solo para el canal nightly.

Los usuarios de .NET pueden utilizar la biblioteca `BenchmarkDotNet` para 
realizar benchmarks de métodos y realizar un seguimiento de su rendimiento. El 
equivalente de `BenchmarkDotNet` es una crate llamada `Criterion`.

Según su [documentación][criterion-docs], `Criterion` recopila y almacena 
información estadística de ejecución en ejecución y puede detectar 
automáticamente regresiones de rendimiento, así como medir optimizaciones.

Usando `Criterion`, es posible utilizar el atributo `#[bench]` sin necesidad de 
cambiar al canal nightly.

Al igual que en `BenchmarkDotNet`, también es posible integrar los resultados de 
los benchmarks con la 
[GitHub Action para Benchmarking Continuo][gh-action-bench]. De hecho, 
`Criterion` admite múltiples formatos de salida, entre los que también se 
encuentra el formato `bencher`, que imita los benchmarks nightly de `libtest` y 
es compatible con la acción mencionada anteriormente.

[cargo-bench]: https://doc.rust-lang.org/cargo/commands/cargo-bench.html
[bench-unstable]: https://doc.rust-lang.org/rustc/tests/index.html#test-attributes
[criterion-docs]: https://bheisler.github.io/criterion.rs/book/index.html
[gh-action-bench]: https://github.com/benchmark-action/github-action-benchmark

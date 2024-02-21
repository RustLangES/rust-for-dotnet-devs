# Herencia

Como se explica en la sección [Estructuras], Rust no proporciona herencia (basada
en clases) como en C#. Una forma de proporcionar un comportamiento compartido 
entre structs es mediante el uso de traits. Sin embargo, similar a la 
_herencia de interfaces_ en C#, Rust permite definir relaciones entre traits 
mediante el uso de [_supertraits_][supertrait.rs].

[Estructuras]: ./custom-types/structs.md
[supertrait.rs]: https://rustlanges.github.io/rust-book-es/ch19-03-advanced-traits.html?highlight=supertrai#usando-supertraits-para-requerir-la-funcionalidad-de-un-trait-dentro-de-otro-trait

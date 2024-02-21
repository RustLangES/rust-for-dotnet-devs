# Namespaces

En .NET, se utilizan namespaces para organizar tipos, así como para controlar 
el ámbito de los tipos y métodos en proyectos.

En Rust, el término "namespace" se refiere a un concepto diferente. El 
equivalente de un namespace en Rust es un [módulo][rust-module]. Tanto en C# 
como en Rust, la visibilidad de los elementos se puede restringir mediante 
modificadores de acceso o modificadores de visibilidad, respectivamente. 
En Rust, la visibilidad predeterminada es _privada_ (con solo algunas 
excepciones). El equivalente al `public` de C# es `pub` en Rust, y `internal` 
se corresponde con `pub(crate)`. Para un control de acceso más detallado, 
consulta la referencia de [modificadores de visibilidad].

[rust-module]: https://doc.rust-lang.org/reference/items/modules.html
[modificadores de visibilidad]: https://doc.rust-lang.org/reference/visibility-and-privacy.html

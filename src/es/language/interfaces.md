# Interfaces

Rust no tiene interfaces como las que se encuentran en C#/.NET. En su lugar, 
tiene _[traits]_. Similar a una interfaz, un trait representa una abstracción y 
sus miembros forman un contrato que debe cumplirse cuando se implementa en un 
tipo.

Al igual que las interfaces pueden tener métodos predeterminados en C#/.NET 
(donde se proporciona un cuerpo de implementación predeterminado como parte de 
la definición de la interfaz), los traits en Rust también pueden tenerlos. El 
tipo que implementa la interfaz/trait puede proporcionar posteriormente una 
implementación más adecuada y/o optimizada.

Las interfaces en C#/.NET pueden tener todo tipo de miembros, desde propiedades,
indexadores y eventos hasta métodos, tanto estáticos como de instancia. De 
manera similar, los traits en Rust pueden tener métodos (de instancia), 
funciones asociadas (piensa en métodos estáticos en C#/.NET) y constantes.

Además de las jerarquías de clases, las interfaces son un medio fundamental para
lograr el polimorfismo mediante la despacho dinámico para abstracciones 
transversales. Permiten escribir código de propósito general contra las 
abstracciones representadas por las interfaces sin tener en cuenta mucho los 
tipos concretos que las implementan. Lo mismo se puede lograr con los 
_Trait Objects_ en Rust de manera limitada. Un trait object es esencialmente una
_v-table_ identificada con la palabra clave `dyn` seguida del nombre del trait, 
como en `dyn Shape` (donde `Shape` es el nombre del trait). Los trait objects 
siempre viven detrás de un puntero, ya sea una referencia (por ejemplo, 
`&dyn Shape`) o el `Box` asignado en el montón (por ejemplo, `Box<dyn Shape>`). 
Esto es algo similar a en .NET, donde una interfaz es un tipo de referencia, de 
modo que un tipo de valor convertido a una interfaz se coloca automáticamente en
la montón gestionado. La limitación de paso de los trait objects mencionada 
anteriormente es que el tipo de implementación original no se puede recuperar. 
En otras palabras, mientras que es bastante común hacer un 
_downcast_ o probar si una interfaz es una instancia de alguna otra interfaz o 
tipo subyacente o concreto, lo mismo no es posible en Rust (sin esfuerzo y 
soporte adicionales).

> Cuando hablamos de downcasting nos referimos al poder obtener a base de una 
> abstracción un tipo concreto.

Puedes mirar también:

- [Traits: Definiendo Comportamiento Compartido - El Lenguaje de Programación Rust][traits]
- [Downcast Trait Object]
- [Downcasting in Rust]

    [traits]: https://book.rustlang-es.org/ch10-02-traits
    [Downcast Trait Object]: https://bennett.dev/rust/downcast-trait-object/
    [Downcasting in Rust]: https://ysantos.com/blog/downcast-rust
# Introducción

Esta es una guía (no exhaustiva) para desarrolladores de C# y .NET que son 
completamente nuevos en el lenguaje de programación Rust. Algunos conceptos y 
construcciones se traducen bastante bien entre C#/.NET y Rust, pero pueden 
expresarse de manera diferente, mientras que otros son un cambio radical, 
como la gestión de la memoria. Esta guía ofrece una breve comparación y mapeo 
de esas construcciones y conceptos con ejemplos concisos.

Los autores[^authors] originales de esta guía eran desarrolladores C#/.Net que 
eran completamente nuevos en Rust. Esta guía es la compilación de conocimiento 
adquirido por los autores escribiendo código Rust durante varios meses. Es la 
guía que los autores desearían haber tenido cuando comenzaron su viaje en Rust.
Dicho esto, los autores te animarían a leer libros y otro material disponible 
en la web para abrazar Rust y sus convenciones en lugar de intentar aprenderlo 
exclusivamente a través del prisma de C# y .NET. Mientras tanto, esta guía 
puede ayudar a responder algunas preguntas rápidamente, como: _¿Rust soporta 
herencia, concurrencia, programación asíncrona, etc.?_

Suposiciones:

- El lector es un experimentado desarrollador de C#/.NET.
- El lector es completamente nuevo en Rust.

Objetivos:

- Proporcionar una breve comparación y mapeo de varios temas de C#/.NET con sus 
contrapartes en Rust.
- Proporcionar enlaces a referencias de Rust, libros y artículos para una 
lectura adicional sobre los temas.

No objetivos:

- Discusión de patrones de diseño y arquitecturas.
- Tutorial sobre el lenguaje Rust.
- Que el lector sea competente en Rust después de leer esta guía.
- Aunque hay ejemplos cortos que contrastan el código de C# y Rust para algunos 
temas, esta guía no pretende ser un recetario de recetas de codificación en los 
dos lenguajes.

---
[^authors]: Los autores originales de esta guía fueron (en orden alfabético):
[Atif Aziz], [Bastian Burger], [Daniele Antonio Maggio], [Dariusz Parys] and
[Patrick Schuler].

  [Atif Aziz]: https://github.com/atifaziz
  [Bastian Burger]: https://github.com/bastbu
  [Daniele Antonio Maggio]: https://github.com/danigian
  [Dariusz Parys]: https://github.com/dariuszparys
  [Patrick Schuler]: https://github.com/p-schuler

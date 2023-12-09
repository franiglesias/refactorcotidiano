# Lecturas recomendadas

> Donde se exponen algunos libros, artículos y vídeos que pueden ser un buen recurso para aprender de autores más cualificados y prestigiosos que el de este libro, cosa que tampoco es tan difícil de encontrar.

## Libros recomendados

[Five Lines of Code](https://www.manning.com/books/five-lines-of-code), de Christian Clausen. Soy consciente de que el prólogo por el _inefable_ Robert C. Martin puede echar para atrás a más de una, pero el libro de Clausen merece una lectura. El autor propone una metodología de refactoring sencilla y progresiva. Parte de una serie de reglas, como la que da título al libro entre otras, para llevarnos de la mano aplicando distintos refactors seguros, por lo que no sería necesario disponer de tests. Con esto, nos ayuda a entender cuál es problema de ese código, por qué y cómo arreglarlo. Para ello nos propone un par de proyectos en TypeScript, lenguaje que toma elementos de otros muchos lenguajes, por lo que no es muy difícil realizar los ejercicios propuestos y llevarlos a tus propios proyectos. 

[Refactoring](https://refactoring.com), de Martin Fowler y Kent Beck. Se trata de un libro de referencia. Es difícil de leer de un tirón porque buena parte de su contenido es un catálogo de técnicas, pero el capítulo sobre _code smells_ es posiblemente el más iluminador de todos. Existen dos ediciones de este libro. En la primera, el refactoring era un concepto todavía en exploración y hay algunos apéndices en los que se trata esa cuestión. En la segunda edición, los ejemplos están en Javascript, lo que posiblemente los hace más accesibles.

[99 bottles of OOP](https://sandimetz.com/99bottles), de Sandi Metz, no es estrictamente un libro sobre refactoring porque su objetivo es enseñar diseño orientado a objetos. Sin embargo, plantea el proceso comenzando por un ejemplo procedural y nos guía en el camino de convertirlo en un programa orientado a objetos, paso a paso y sin romper la funcionalidad. Creo que el término que mejor encaja para eso es _refactoring_. Además, tienes una edición del libro para PHP, Ruby y JS. Sandi Metz es una de mis autoras favoritas, así que recomiendo a ciegas cualquier cosa que publique, y sus charlas son fantásticas.

## Blogs y otros recursos online

[Refactoring.guru](https://refactoring.guru/refactoring) es un sitio web que, en parte, reproduce muchos elementos de los libros de M. Fowler, algo que en su día les llevó a ciertos problemas relacionados con la propiedad intelectual. El caso es que esta web nos presenta un catálogo de refactorings y code smells muy apañado, con numerosos ejemplos, explicaciones, esquemas e ideas. La verdad es que mola bastante.


[The talking bit](https://franiglesias.github.io/tag/refactoring/). El origen de este libro es mi blog personal, que está bastante lleno de contenido técnico relacionado sobre todo con testing, diseño de software y, por supuesto, refactoring. Y está en español, que de recursos en inglés estamos surtidas.

## Vídeos

[Workflows of refactoring](https://youtu.be/vqEg37e4Mkw?si=csOr06PlyZxC_0L-) es la charla de Martin Fowler en la que explica cuando se hace el refactoring.

[The real secret to refactoring](https://youtu.be/fx-Ne_s71iY?si=mreVP0fvSoLchBuR) siempre merece la pena seguir el material de David Farley.

[Refactoring práctico](https://youtube.com/playlist?list=PLYT8quZ2BEnZzUZ9c1mIQyAupob9fZeFj&si=GPN1p6lHUvrisDnr) _The Talking Bit_ tiene un canal en Youtube, en el que voy poniendo, cuando me siento suficientemente inspirado y tengo silencio en casa, vídeos en los que explico procesos de refactoring, TDD, etc. Son un apoyo al material publicado en el blog porque en algún momento pensé que los artículos escritos no conseguían transmitir el proceso sobre el terreno. Como punto característico, no se trata precisamente de _shorts_, prácticamente son sesiones de _live coding_ (a veces corto algún trocito o edito algún error), por lo que tienen una duración larga. Básicamente, soy yo haciendo ejercicios y pensando en voz alta. 

[Refactoring con calisthenics](https://youtube.com/playlist?list=PLYT8quZ2BEnbKznU9jxeswl8IEEeMSJFL&si=M1r1KrQehUYZj1-H) Esta es otra serie de vídeos en las que aplico las reglas de calistenia de Jeff Way como forma de dirigir el refactor de un código. En concreto de un ejemplo adaptado por Emily Bache, a partir de otro puesto por Fowler en el libro de Refactoring.

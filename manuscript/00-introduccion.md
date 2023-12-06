# Introducción de la primera edición

En The Talking Bit hemos [escrito bastante sobre refactor](https://franiglesias.github.io/tag/refactoring/), principalmente porque nos parece que es una de las mejores cosas que podemos hacer con nuestro código, sea nuevo o legacy. El refactor es una forma de mantenimiento del código que consiste en mejorar su expresividad a través de pequeños cambios que no alteran el comportamiento y que tampoco cambian sustancialmente la implementación.

Una reescritura, por el contrario, suele plantear un cambio brusco de implementación que podría incluso provocar cambios en el comportamiento. Este proceso busca reconstruir el software a partir de nuevos principios o ideas.

Por otro lado, el refactor puede hacerse de una manera continua e iterativa, interviniendo en el código siempre que se tenga ocasión. Por ejemplo: porque estamos revisándolo a fin de corregir un error o añadir una nueva característica.

Durante el proceso de lectura y análisis podemos encontrarnos con fragmentos de código que no expresan bien un concepto del dominio, que no se entienden fácilmente o que interfieren en la compresión de ese código. Ese momento es ideal para realizar pequeños refactors seguros que, acumulados a lo largo del tiempo, van haciendo que el código evolucione hacia un mejor diseño. Pero sobre todo, hacen que el código refleje cada vez mejor el conocimiento que tenemos.

Como ya hemos mencionado en otras ocasiones, [el refactor trata principalmente sobre conocimiento y significado](https://franiglesias.github.io/refactor-knowledge-meaning/). Es decir, trata sobre que el código exprese cosas y, concretamente, que exprese de la mejor forma posible nuestro conocimiento sobre el dominio en el que trabajamos y cómo estamos resolviendo los problemas que nos plantea.

Por esa razón, se nos ha ocurrido que podría ser buena idea crear una especie de curso o guía para aprender sobre cómo hacer _refactor cotidiano_.

## Refactor sin piedad

El refactor cotidiano es otro nombre para una práctica de _Extreme Programming_ conocida como _refactor sin piedad_. Dicho en pocas palabras, consiste en refactorizar el código en cualquier momento en que sea posible. Contra lo que podría parecer no se trata de engolfarse dando vueltas al código hasta llegar a un código perfecto, sino de realizar retoques limitados para corregir lo que en ese momento podemos considerar como pequeñas imperfecciones. Por ejemplo:

* Un nombre no consigue expresar correctamente un concepto.
* Una expresión compleja podría descomponerse otras más pequeñas, más fáciles de entender y mantener.
* El cuerpo de un método o función es muy largo y puede ser dividido en secciones autónomas.
* Una expresión condicional es difícil de entender y se puede encapsular como función o método con un nombre expresivo, ocultando sus detalles.
* Una estructura condicional presenta demasiada anidación, de modo que extraemos sus partes para organizarlo según niveles de abstracción.
* Un fragmento de código está en lugar inadecuado y lo movemos a donde corresponde.

Con el tiempo, la acumulación de estos pequeños cambios irá mejorando la estructura, expresividad y mantenibilidad del código. Es muy posible que acabe revelando oportunidades de cambio de mayor calado que tal vez impliquen un rediseño.

En cualquier caso, al mejorar la situación del código, mejoran nuestras posibilidades de intervenir en él. Incorporar nuevas prestaciones será más rápido y seguro, lo que nos permite entregar con mayor frecuencia y predictibilidad.

## Los momentos del refactor

En su charla _Flows of refactoring_, Martin Fowler explica qué es realmente el refactor y cuándo se tendría que hacer.

Fowler insiste en la idea de que si necesitas un momento especial para refactorizar es que no lo estás haciendo, sino, tal vez, reescribiendo o cambiando el diseño de tu software.

Existen tres momentos principales para el refactor:

* Durante la lectura de código
* Refactor preparatorio para introducir un cambio
* Refactor posterior a un cambio

### Lectura de código, los momentos WTF

Pasamos la mayor parte del tiempo leyendo código. Cada vez que queremos introducir un cambio necesitamos leer el código existente para saber dónde es el mejor lugar para hacerlo o averiguar qué recursos ya están disponibles. También leemos el código simplemente para buscar algún tipo de conocimiento relacionado con el código mismo o el negocio.

En esos momentos podemos encontrarnos con fragmentos que nos hagan lanzar una interjección (WTF!), o que necesitamos leer dos o tres veces para entender lo que ocurre. Momentos en los que vemos algo que, por alguna razón no cuadra. No es que no funcione, es simplemente que hay algo descolocado, sucio, fuera de sitio, desentonando. En una palabra: desordenado.

Por lo general, estos momentos de extrañeza los genera la existencia de una distancia entre nuestro conocimiento del negocio y cómo se expresa en el propio código. Este desfase es lo que Ward Cunningham denominó _deuda técnica_.

La deuda técnica se paga refactorizando el software para que refleje el conocimiento del negocio de la mejor manera posible.

En cierto modo, es imposible evitar que el código tenga un cierto nivel de deuda técnica porque el conocimiento del negocio cambia mucho más rápidamente. De hecho, cuando introducimos cambios en el software y los llevamos a producción, el negocio ya está cambiando. En parte, gracias a las nuevas funcionalidades del software y al impacto que tienen.

La forma de prevenir el crecimiento de la deuda técnica es entregar de forma sostenida pequeños cambios y refactorizar constantemente.

### Refactor preparatorio

Al introducir cambios en el código para incorporar nuevas funcionalidades o corregir errores suele ser necesario arreglar el código existente para hacer posibles esos mismos cambios. Una cita de Kent Beck lo expresa más o menos así:

> For each desired change, make the change easy (warning: this may be hard), then make the easy change

Es decir:

> Primero, haz que el cambio sea fácil (advertencia: puede ser difícil conseguirlo). Luego haz el cambio fácil.

Con frecuencia el código no está preparado para admitir ciertos cambios, por lo que debemos transformarlo primero a fin de que la nueva funcionalidad sea fácil de introducir.

Esto es lo que denominamos refactor preparatorio y debería hacerse siempre antes de introducir los nuevos cambios. En otras palabras: mejoramos la estructura del código asegurándonos de no cambiar su comportamiento. Mezclamos esos cambios y solo entonces procedemos a desarrollar la nueva funcionalidad.

Usando otra metáfora de Kent Beck: usamos dos sombreros: el de refactorizar y el de crear nueva funcionalidad. No podemos llevar los dos sombreros a la vez.

### Refactor posterior a un cambio

Tras realizar los cambios necesarios volvemos a encasquetarnos el sombrero de refactor para limpiar el código que acabamos de introducir.

Es posible que nuestra nueva funcionalidad haya introducido algo de duplicación y esto podría revelar la existencia de un concepto más general que hasta entonces no habíamos descubierto. En otras ocasiones, el código nuevo incrementa la complejidad de algún área y debemos trabajar para simplificarla.

Al añadir código a un sistema de software, aumentamos su entropía o desorden. La entropía de software es un concepto introducido por Ivar Jacobson y otros en el libro _Object-Oriented Software Engineering: A Use Case Driven Approach_. La mejor forma de luchar contra un crecimiento desmesurado de la entropía, es aplicar un refactor continuado, particularmente tras introducir código nuevo.

## Cómo refactorizar

La idea del refactor cotidiano es muy simple:

Se trata de realizar pequeños cambios inocuos en nuestro código en cualquier momento que se nos presente la ocasión. Es lo que algunos autores denominan **refactor oportunista**.

Nuestra propuesta concreta es que hagas un refactor muy pequeño cada vez que lo veas necesario, de modo que, en una primera fase:

* solo tocas un archivo.
* los cambios quedan recogidos en un único commit atómico, que contengan solo los cambios debidos a ese refactor.

En una segunda fase:

* los cambios podrían a varios archivos, pero el ámbito es limitado.
* los cambios quedan recogidos en un único commit atómico.

En una tercera fase:

* los cambios podrían suponer introducción de nuevas clases.
* de nuevo, los cambios quedarían recogidos en un único commit.

## La guía

Esta guía se compone de una serie de capítulos en los que se exponen diversas orientaciones y principios que seguir a la hora de refactorizar. La idea es explicar ámbitos en los que podrías intervenir en los tres niveles indicados en el apartado anterior.

En muchos casos los refactors propuestos, al menos en el primer nivel o fase no necesitarían tests porque podrían ejecutarse mediante herramientas del IDE.

Con el tiempo, es posible que esos pequeños refactors acumulado día tras día, mejoren la forma y calidad de tu código y te despejen caminos para mejorar su expresividad y arquitectura.

Así que, _¡happy tiding!_

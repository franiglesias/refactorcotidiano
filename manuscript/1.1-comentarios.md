# Cuando los comentarios confunden

> En el que se trata de como gestionar los comentarios de un código, con especial atención a las situaciones en que el comentario más que ayudar nos confunde, así como criterios para decidir qué comentarios mantener y cuáles no.

## Comentarios y documentación

Los lenguajes de programación incluyen la posibilidad de insertar comentarios en el código como forma de añadir documentación al mismo. Es decir: el objetivo de los comentarios es poner conocimiento cerca del lugar en el que puede ser necesario.

Los comentarios en el código parecen una buena idea y, probablemente, eran más útiles en otros tiempos, cuando la necesidad de economizar recursos y las limitaciones de los lenguajes de programación no nos permitían escribir un código lo bastante expresivo como para ser capaz de documentarse a sí mismo.

Así que, en este capítulo, intentaremos explicar qué comentarios nos sobran y por qué, y cuáles dejar.

### Notas de la segunda edición

No hay grandes cambios en este capítulo. En general, se mantienen las mismas ideas que en la primera edición. Lo más significativo es que hemos añadido una sección con una heurística (las seis preguntas) que puede ser útil cuando tengas que decidir si añadir o mantener un comentario.

Por lo demás, este capítulo no es una diatriba general contra los comentarios o la documentación en el código, sino contra la documentación innecesaria, desactualizada o aquella que puede quedar fácilmente desactualizada.

Como norma general, la documentación debería estar lo más cerca posible del propio código. Esta podría ser una forma útil de organizar la documentación:

* El código expresa claramente qué hace y cómo.
* Los tests documentan cómo se usa el código, mediante ejemplos que se pueden ejecutar.
* Se añaden comentarios cuando clarifican el porqué de ciertas decisiones.
* El gestor de versiones documenta la historia del desarrollo.
* El archivo README explica la naturaleza del proyecto, su relación con otros proyectos en su caso, y documenta el proceso de instalación, uso en local, desarrollo y testing.
* Mejor aún si estos procesos están automatizados, mediante una herramienta tipo `make` o la equivalente para un ecosistema específico. Considérala una _pipeline_ local.
* El archivo README enlaza a documentos que puedan ser útiles, como how-to, configuración del IDE para el proyecto, o cualquier otro proceso que se considere oportuno, y se guardan en el mismo repositorio.
* Las decisiones más abstractas sobre el código, como pueden ser decisiones sobre diseño, convenciones de código, decisiones sobre tecnologías, etc., se pueden documentar mediante Architecture Decision Records (ADR), que se guardarán también en el repositorio.
* Los manuales de uso de la aplicación o el software pueden documentarse externamente, pero es conveniente enlazarlo en el README del proyecto, que actuaría como punto de entrada.

## ¿Por qué deberías eliminar comentarios?

Las principales razones para borrar comentarios son:

**Suponen una dificultad añadida para leer el código**. En muchos aspectos, los comentarios presentan una narrativa paralela a la del código y nuestro cerebro tiende a enfocarse en una de las dos. Si nos enfocamos en la de los comentarios, no estamos leyendo el código. Si nos enfocamos en la del código: ¿para qué queremos comentarios?

**Los comentarios suponen una carga cognitiva**. Incluso leyéndolos con el rabillo del ojo, los comentarios pueden suponer una carga cognitiva si, de algún modo, discrepan con lo que el código dice. Esto puede interrumpir tu flujo de lectura hasta que consigues aclarar si ese comentario tiene algún valor o no.

**Pueden alargar innecesariamente un bloque de código**. Idealmente, deberías poder leer un bloque de código en una sola pantalla. Los comentarios añaden líneas que podrían provocar que tengas que deslizar para ver todo el bloque. Son especialmente problemáticos los que están intercalados con las líneas de código.

**Pueden mentir**. Con el tiempo, si no se hace mantenimiento de los comentarios, estos acaban siendo mentirosos. Esto ocurre porque los cambios en el código no siempre son reflejados con cambios en los comentarios por lo que llegará un momento en que unos y otros no tengan nada que ver.

## Refactor de comentarios

### Básico

Simplemente, eliminamos los comentarios que no necesitamos. Es un refactor completamente seguro, ya que no afecta de ningún modo al código.

### Reemplazar comentarios por mejores nombres

Eliminamos comentarios obvios, redundantes o innecesarios, cambiando el nombre de los símbolos que tratan de explicar.

**Cambiar nombres** es un refactor muy seguro, sobre todo con la ayuda de un buen IDE, que puede realizarlo automáticamente, y dentro de ámbitos seguros, como método o variables privadas.

### Reemplazar comentarios por nuevas implementaciones

En algunos casos podríamos plantearnos mejorar el diseño de una parte del código porque al reflexionar sobre la necesidad de mantener un comentario nos damos cuenta de que es posible expresar la misma idea en el código.

Este tipo de refactor no encaja en la idea de esta serie sobre refactor cotidiano, pero plantea el modo en que los pequeños refactors del día a día van despejando el camino para refactors e incluso reescrituras de mayor alcance.

### Comentarios redundantes

Los comentarios redundantes son aquellos que nos repiten lo que ya dice el código, por lo que podemos eliminarlos.

Por ejemplo:

```php
// Class to represent a Book
class Book
{
    //...
}
```

En serio, ¿qué nos aporta este comentario que no esté ya expresado?

```php
class Book
{
    //...
}
```

Los lenguajes fuertemente tipados, que soportan *type hinting* y/o *return typing*, nos ahorran toneladas de comentarios. Y de tests.

```php
class Book
{
    /**
    * Creates a book with a title and an author
    *
    * @param Title $title
    * @param Author $author
    * @returns Book
    */
    public static function create(Title $title, Author $author): Book
    {
        //
    }
}
```

Los tipos de los parámetros y del objeto devuelto están explícitos en el código, por lo que es redundante que aparezcan como comentarios.

```php
class Book
{
    /**
    * Creates a book with a title and an author
    */
    public static function create(Title $title, Author $author): Book
    {
        //
    }
}
```


**Excepciones**: este tipo de comentarios tiene su razón de ser cuando no podemos hacer explícitos los tipos.

Puedes eliminar los comentarios redundantes poniendo mejores nombres. Por ejemplo, en este caso en que utilizamos un constructor secundario:

```php
class Book
{
    /**
    * Creates a book with a title and an author
    */
    public static function create(Title $title, Author $author): Book
    {
        //
    }
}
```

Con un nombre expresivo ya no necesitamos comentario:

```php
class Book
{
    public static function withTitleAndAuthor(Title $title, Author $author): Book
    {
        //
    }
}
```

O incluso más explícito:

```php
class Book
{
    public static function newWithTitleAndAuthor(Title $title, Author $author): Book
    {
        //
    }
}
```

Y podemos usar el objeto así, lo cual documenta perfectamente lo que está pasando:

```php
$newBook = Book::withTitleAndAuthor($title, $author);
```

```php
$newBook = Book::newWithTitleAndAuthor($title, $author);
```

**Más excepciones**: si lo que estamos desarrollando es una librería que pueda utilizarse en múltiples proyectos, incluso que no sean nuestros, los comentarios que describen lo que hace el código pueden ser necesarios.

### Comentarios mentirosos

Los comentarios mentirosos son aquellos que dicen algo distinto que el código. Y, por tanto, deben desaparecer.

¿De dónde vienen los comentarios mentirosos? Sencillamente, ha ocurrido que los comentarios se han quedado olvidados, sin mantenimiento, mientras que el código ha evolucionado. Por eso, cuando los lees hoy es posible que digan cosas que ya no valen para nada.

Mi experiencia personal con este tipo de comentarios cuando entro a un código nuevo suele ser bastante negativa. Si el comentario y el código difieren te encuentras con el problema de decidir a cuál de los dos hacer caso. Lo cierto es que el código manda porque es lo que realmente se está ejecutando y lo que está produciendo resultados, pero la existencia del comentario genera esa inquietud: ¿por qué el comentario dice una cosa y el código hace otra?

Habitualmente, debería ser suficiente con comprobar si hay diferencias en la fecha en que se añadió el comentario y la fecha en la que se modificó el código. Si esta es posterior, es que tenemos un caso de comentario mentiroso por abandono y lo más adecuado sería borrarlo. 

Este hecho debería bastar para que no añadas nuevos comentarios sin una buena razón. Tendemos a ignorar los comentarios triviales, de modo que cuando cambiamos el código nos despreocupamos de mantenerlos actualizados y acaban siendo mentirosos. Así que procuraremos dejar solo aquellos comentarios que nos importen realmente.

Si ya nos hemos librado de los comentarios redundantes, deberíamos contar solo con los que pueden aportar alguna información útil, así que nos toca examinarlos para asegurarnos de que no sean mentirosos. Y serán mentirosos si no nos cuentan lo mismo que cuenta el código.

Puede parecer un poco absurdo, pero al fin y al cabo los comentarios simplemente están ahí y no les prestamos mucha atención, salvo que sea la primera vez que nos movemos por cierto fragmento de código y tratamos de aprovechar cualquier información que nos parezca útil. Es entonces cuando descubrimos comentarios que pueden oscilar entre lo simplemente desactualizado y lo esperpéntico.

Así que, fuera con ellos. Algunos ejemplos:

**To-dos olvidados**. Las anotaciones **To do** seguramente hace meses que han dejado de tener sentido. Mienten en tanto que no tenemos ninguna referencia que les aporte significado. 

¿De qué otro tipo de servicio estábamos hablando aquí hace tres meses? ¿Será que ya lo hemos cambiado?

```php
// @todo we should use another kind of service here

$service = new Service();
$service->execute();
```

Sería diferente si el comentario fuese mucho más preciso y detallado, de tal forma que indique con claridad el ámbito y plazo de la tarea pendiente. Algo así:

```php
/**
@todo we should replace this service with the new implementation that support Kafka brokers when infra team finished the migration from SQS, scheduled for 18/09/2023
*/

$service = new Service();
$service->execute();
```

En ese caso, el comentario hace explícitos unos detalles que definen con precisión los motivos, acciones y plazos.

**Comentarios olvidados**. En algunos casos puede ocurrir que simplemente nos hayamos dejado comentarios olvidados. Por ejemplo, podríamos haber usado comentarios para definir las líneas básicas de un algoritmo, que es una técnica bien conocida, y ahí se habrían quedado. Todo ello también tiene que desaparecer:

```php
public function calculateFee(Request $dataToCalculate)
{
    // Normalize amounts to same currency
    
    // ... code here
    
    // Perform initial calculation
    
    // ... more code here
    
    // Apply transormation
    
    // .., more code here
}
```

**Comentarios para estructurar código**. Claro que puede que el algoritmo sea lo bastante complejo como para que necesitemos describir sus diferentes partes. En este caso, el mejor refactor es extraer esas partes a métodos privados con nombres descriptivos, en lugar de usar comentarios:

```php
public function calculateFee(Request $dataToCalculate)
{
    $this->normalizeAmountsToTheSameCurrency($dataToCalculate);
    $initialCalculation = $this->performInitialCalculation($dataToCalculate);
    $transformedResponse = $this->applyTransformation($initialCalculation);
}
```

De este modo el código está estructurado y documentado.

**Comentarios sobre valores válidos**. Consideremos este código:

```php
// Valid values: started, paused, running, terminated
public function updateStatus(string $newStatus): void
{
    $this->checkValidStatus($newStatus);
    $this->status = $newStatus;
}
```

El comentario delimita los valores aceptables para un parámetro, pero no fuerza ninguno de ellos. Eso tenemos que hacerlo mediante una cláusula de guarda. ¿Hay una forma mejor de hacerlo?

Por supuesto: utilizar un _enumerable_.

```php
class Status
{
    private const STARTED = 'started';
    private const PAUSED = 'paused';
    private const RUNNING = 'running';
    private const TERMINATED = 'terminated';
    
    private $value;
    
    private function __construct(string $status)
    {
        $this->checkValidStatus($status);
        $this->status = $status;    
    }
    
    public static function fromString(string $status): Status
    {
        return new self($status);
    }
    
    public static function started(): Status
    {
        return new self(self::STARTED);
    }
    
    //...
}
```

Lo que permite eliminar el comentario, a la vez que tener una implementación más limpia y coherente:

```php
public function updateStatus(Status $newStatus): void
{
    $this->status = $newStatus;
}
```

### Código comentado

En alguna parte he escuchado o leído algo así como "código comentado: código borrado". El código comentado debería desaparecer. Lo más seguro es que ya nadie se acuerde de por qué estaba ese código ahí, para empezar, y por qué sigue aunque sea escondido en un comentario.

Si es necesario recuperarlo (spoiler: no lo será) siempre nos queda el control de versiones.

**Excepciones**: a veces se puede usar la técnica de comentar un código para desactivarlo temporalmente. En ese caso, deberíamos explicar esa decisión también en el mismo comentario. Mucho mejor que eso es utilizar alguna técnica de _feature flag_. Existen librerías en todos los lenguajes para gestionar _feature flags_, pero en muchos casos podemos introducer alguna variable que sea fácil de cambiar:

```php
if ($useNewCode === true) {
    $this->newCode();
} else {
    $this->oldCode();
}
```

## Comentarios que podríamos conservar… o no

### Comentarios que explican decisiones

Los buenos comentarios deberían explicar por qué tomamos alguna decisión que no podemos expresar mediante el propio código y que, por su naturaleza, podríamos considerar como independientes de la implementación concreta que el código realiza. Es decir, no deberíamos escribir comentarios que expliquen cómo es el código, que es algo que ya podemos ver, sino que expliquen por qué es así.

Lo normal es que estos comentarios sean pocos, pero relevantes, lo cual los pone en una buena situación para realizar un mantenimiento activo de los mismos.

Obviamente, corremos el riesgo de que los comentarios se hagan obsoletos si olvidamos actualizarlos cuando sea necesario. Por eso la importancia de que no estén "acoplados" a la implementación en código.

Un ejemplo de comentario relevante podría ser este:

```php

// We apply taxes to conform the procedure stated in law RD 2018/09
public function applyTaxes(Money $totalAmountBeforeTaxes): Money
{
    //... some code here
}
```

Este comentario es completamente independiente del código e indica una información importante que no podríamos expresar con él. Si en un momento dado cambia la legislación y debemos aplicar otra normativa, podemos cambiar el comentario.

Aunque, a decir verdad, podríamos llegar a expresarlo en código. A grandes rasgos:

```php

interface Taxes
{
    public function apply(Money $amountBeforeTaxes): Money;
}

class RD201809Taxes implements Taxes
{
    public function apply(Money $amountBeforeTaxes): Money
    {
        // ... some code here
    }
}


class RD201821Taxes implements Taxes
{
    public function apply(Money $amountBeforeTaxes): Money
    {
        // ... some code here
    }
}
```

## Dudas razonables

### Comentarios para el IDE

En aquellos lenguajes en los que el análisis estático por parte del IDE no pueda interpretar algunas cosas, añadir comentarios en forma de anotaciones puede suponer una ayuda para el IDE. En algunos casos, gracias a eso nos avisa de problemas potenciales antes de integrar los cambios.

No debería ser una práctica común, pero es un compromiso aceptable. Por ejemplo, en PHP era frecuente indicar el tipo de las propiedades de los objetos y otras variables con comentarios, ya que el lenguaje no permitía hacerlo en código. 

```php
class Status
{
    /** string **/
    private $value;
}
```

Esto se introdujo en la versión 7.4:

```php
class Status
{
    private string $value;
}
```

En otros lenguajes, estas características estaban presentes desde mucho antes.

## Las seis preguntas

El framework de las seis preguntas se utiliza en algunas disciplinas para determinar si una fuente proporciona información completa. Estas preguntas se pueden usar para decidir qué comentar en un código y qué no es necesario:

* **¿Cuándo se ha escrito el código?**: Esta información la encontramos fácilmente en el sistema de control de versiones. No hay que añadirla como comentario. El único caso en que se me ocurre que podría ser útil es cuando mudamos un repositorio a un servidor diferente, ya que se puede perder la información.
* **¿Quién ha escrito el código?**: Aplica lo mismo que en la pregunta anterior, es información que nos proporciona el sistema de control de versiones, de una forma mucho más precisa.
* **¿Dónde está el código?**: Aplicado a la _paquetización_ del código, básicamente es una información que o bien se declara de forma explícita, o bien el lenguaje se encarga de reportarnos en caso de errores. Por tanto, tampoco parece necesario establecerlo en un comentario.
* **¿Qué hace el código?**: La respuesta corta es que el código ya dice lo que hace, pero con frecuencia eso no queda tan claro porque los nombres están mal escogidos o la estructura del código lleva a confusión. Eso podría llevarnos a plantear la necesidad de indicarlo en un comentario. Pero antes de ellos, lo apropiado sería reflexionar sobre cómo explicitar la intención de ese fragmento de código usando un buen nombre. Y, de todos modos, la mejor forma de documentar lo que hace un código es mediante un test.
* **¿Cómo hace el código lo que hace?**: De nuevo, una vez que sabemos lo que hace un código, el cómo debería ser el código en sí. Ahora bien, hay algunos casos en los que es recomendable añadir comentarios. Uno de esos casos es el uso de algoritmos bien conocidos, que tienen nombre. En esa situación, es muy buena idea hacerlo explícito. Otro caso podría ser el de documentar distintos pasos en un algoritmo, aunque para ello suele ser mejor extraerlos a sus propios métodos.
* **¿Por qué hace el código lo que hace?**: Finalmente, esta es una pregunta que solo podemos contestar nosotras: las personas responsables de ese conocimiento. Y esa explicación debe aparecer como comentario.

## Resumen del capítulo

Los comentarios en el código tienen una utilidad limitada y, con frecuencia, se vuelven mentirosos y no resultan de ayuda para comprender lo que nuestro código hace, pudiendo incluso llevarnos a confusión si les hacemos caso.

Si introduces un comentario, debes responsabilizarte de su ciclo de vida: actualizarlo cuando cambie el código al que hace referencia. Borrarlo si ya no sirve de nada.

En lugar de usar comentarios es preferible trabajar en mejores nombres para los símbolos (variables, constantes, clases, métodos, funciones…), estructurar mejor el código en funciones o métodos que expresen su intención.

Si necesitamos documentar cómo funciona algo y cómo usarlo, es mucho mejor introducir tests, los cuales proporcionan una documentación viva.

Por otro lado, los comentarios que sí pueden permanecer suelen referirse a aspectos que no podemos expresar fácilmente con código, como puede ser explicar los motivos para hacer algo de una forma concreta. Aún así, debes asegurarte de mantenerlos al día.

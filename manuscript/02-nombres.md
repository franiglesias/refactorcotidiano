# Capítulo 2. El nombre de la cosa

> En el que Adso y Guillermo... ejem, en el que discutimos sobre la necesidad de poner los nombres adecuados a las cosas del código, a fin de que se entienda qué demonios pasaba en aquel momento por nuestra cabeza, o la de la autora del código que tenemos que intervenir. 

Probablemente en ningún lugar como el código los nombres configuran la realidad. Escribir código implica establecer decenas de nombres cada día, para identificar conceptos y procesos. Una mala elección de nombre puede condicionar nuestra forma de ver un problema de negocio. Un nombre ambiguo puede llevarnos a entrar en un callejón sin salida, ahora o en un futuro no muy lejano. Pero un nombre bien escogido puede ahorrarnos tiempo, dinero y dificultades.

### Notas de la segunda edición

En este capítulo hemos cambiado los ejemplos para que los nombres originales sean mucho menos expresivos. De este modo, se entiende mejor la necesidad del refactor, y también se entiende mejor lo que hace la clase `Calculator` que hemos usado como ejercicio.

## Símbolos con nombres

Un trozo de código debería poder leerse como una especie de narrativa, en la cual cada palabra expresase de forma unívoca un significado. También de forma ubicua y coherente, es decir, que el mismo símbolo debería representar el mismo concepto en todas partes del código.

## ¿Cuándo refactorizar nombres?

La regla de oro es muy sencilla: cada vez que al leer una línea de código tenemos que pararnos a pensar qué está diciendo lo más probable sea que debamos cambiar algún nombre.

Este es un ejemplo de un código en el que nos encontramos con unos cuantos problemas de nombres, algunos son evidentes y otros no tanto:

```php
class Item {
    private $val;

    public function __construct($val) {
        $this->val = $val;
    }

    public function getVal() {
        return $this->val;
    }
}

class Calculator {
    private $rate1;
    private $rate2;

    public function __construct($rate1, $rate2) {
        $this->rate1 = $rate1;
        $this->rate2 = $rate2;
    }

    public function calc($item) {
        $val = $item->getVal();
        $val = $this->applyRate2($val);
        $val = $this->applyRate1($val);
        return $val;
    }

    private function applyRate2($val) {
        return $val * (1 + $this->rate2);
    }

    private function applyRate1($val) {
        return $val * (1 - $this->rate1);
    }
}
```

Por supuesto, en este ejemplo hay algunos errores más aparte de los nombres. Pero hoy solo nos ocuparemos de estos. Vamos por partes.

### Nombres demasiado genéricos

Los nombres demasiado genéricos requieren el esfuerzo de interpretar el caso concreto en que se están aplicando. Además, en un plano más práctico, resulta difícil localizar una aparición específica del mismo que tenga el significado deseado.

¿De dónde vienen los nombres demasiado genéricos? Normalmente, vienen de estadios iniciales del código, en los que probablemente bastaba con ese término genérico para designar un concepto. Con el tiempo, ese concepto evoluciona y se ramifica a medida que el conocimiento de negocio avanza, pero el código puede que no lo haya hecho al mismo ritmo, con lo que llega un momento en que este no es reflejo del conocimiento actual que tenemos del negocio.

_Calculate… what?_ Exactamente, ¿qué estamos calculando aquí? El código no lo refleja. Podría ocurrir, por ejemplo, que `$rate1` fuese algún tipo de descuento, `$rate2` podría ser una comisión o impuestos y `$val` parece claro que es algo así como el precio de tarifa de algún producto o servicio, sea lo que sea que vende esta empresa. Es muy posible que este método lo que haga sea calcular el precio final para el consumidor del producto. ¿Por qué no declararlo de forma explícita?

```php
class Calculator {
    private $rate1;
    private $rate2;

    public function __construct($rate1, $rate2) {
        $this->rate1 = $rate1;
        $this->rate2 = $rate2;
    }

    public function finalPrice($item) {
        $val = $item->getVal();
        $val = $this->applyRate2($val);
        $val = $this->applyRate1($val);
        return $val;
    }

    private function applyRate2($val) {
        return $val * (1 + $this->rate2);
    }

    private function applyRate1($val) {
        return $val * (1 - $this->rate1);
    }
}
```

Vamos a revisar los distintos nombres que se están usando en el código para representar los conceptos que se manejan en esta calculadora de precios. Puesto que tenemos bastante claro que `$val` es el precio del producto, podemos hacerlo explícito.

```php
class Calculator {
    private $rate1;
    private $rate2;

    public function __construct($rate1, $rate2) {
        $this->rate1 = $rate1;
        $this->rate2 = $rate2;
    }

    public function finalPrice($item) {
        $price = $item->getVal();
        $price = $this->applyRate2($price);
        $price = $this->applyRate1($price);
        return $price;
    }

    private function applyRate2($price) {
        return $price * (1 + $this->rate2);
    }

    private function applyRate1($price) {
        return $price * (1 - $this->rate1);
    }
}
```

La clase `Item`, que representa el producto o servicio que estamos vendiendo nos proporciona ese precio base e, igualmente, deberíamos hacerlo explícito.

```php

class Item {
    private $price;

    public function __construct($price) {
        $this->basePrice = $price;
    }

    public function basePrice() {
        return $this->basePrice;
    }
}
```

Con estos cambios de nombres, debería haber quedado más claro qué es lo que está pasando.

```php
class Calculator {
    private $rate1;
    private $rate2;

    public function __construct($rate1, $rate2) {
        $this->rate1 = $rate1;
        $this->rate2 = $rate2;
    }

    public function finalPrice($item) {
        $price = $item->basePrice();
        $price = $this->applyRate2($price);
        $price = $this->applyRate1($price);
        return $price;
    }

    private function applyRate2($price) {
        return $price * (1 + $this->rate2);
    }

    private function applyRate1($price) {
        return $price * (1 - $this->rate1);
    }
}
```

### Nombres ambiguos

Hay dos propiedades en la calculadora que tienen el mismo nombre `$rate1` y `$rate2`. 

Técnicamente, son correctos, ya que `$rate` nos sugiere un porcentaje o proporción, algo que podemos confirmar al leer los métodos en los que se aplican. Pero, ¿qué concepto de negocio representan?

Sabemos que se aplican dos, pero no sabemos qué representan. Uno de ellos se resta y el otro se suma. El que se resta, puede ser un descuento, mientras que el que se suma, podría tratarse de un impuesto o una comisión. Debería ser obvio que necesitamos clarificarlo.

Tras hablarlo con negocio, hemos llegado a la conclusión de `$rate1` representa un descuento y `$rate2` un impuesto. Lo adecuado, en este caso, será poner nombres explícitos.

```php
class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $price = $item->basePrice();
        $price = $this->applyRate2($price);
        $price = $this->applyRate1($price);
        return $price;
    }

    private function applyRate2($price) {
        return $price * (1 + $this->tax);
    }

    private function applyRate1($price) {
        return $price * (1 - $this->discount);
    }
}
```

Con este refactor ya hemos ganado mucho en expresividad, pero el término Rate se utiliza en varios nombres compuestos, por lo que no hemos terminado aún. El siguiente paso sería cambiar `applyRate1` y `applyRate2` por algo más descriptivo:

```php
class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $price = $item->basePrice();
        $price = $this->applyTax($price);
        $price = $this->applyDiscount($price);
        return $price;
    }

    private function applyTax($price) {
        return $price * (1 + $this->tax);
    }

    private function applyDiscount($price) {
        return $price * (1 - $this->discount);
    }
}
```

Aplicando este refactor hemos conseguido mucha más claridad y legibilidad. Incluso puede que hayamos hecho aflorar un _bug_. Si te fijas, se aplican los descuentos tras aplicar los impuestos. En muchos países, los impuestos se aplican sobre el precio final, por lo que deberíamos cambiar el orden de aplicación de los métodos `applyTax` y `applyDiscount`.

```php
class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $price = $item->basePrice();
        $price = $this->applyDiscount($price);
        $price = $this->applyTax($price);
        return $price;
    }

    private function applyTax($price) {
        return $price * (1 + $this->tax);
    }

    private function applyDiscount($price) {
        return $price * (1 - $this->discount);
    }
}
```

Esto es muy interesante porque nos demuestra como los nombres ambiguos puede llevarnos a errores de lógica. Al emplear el mismo término `$rate` para referirnos a dos conceptos completamente distintos, hemos podido confundirnos y aplicar los descuentos antes de los impuestos, lo que podría llevar a problemas legales o a pérdidas económicas. Ciertamente, sería posible prevenirlo con un buen test, que nos habría indicado el error cometido, pero, y si no tenemos un test que cubra ese caso concreto, ¿cómo lo detectaríamos?

Pero es que, además, al eliminar la ambigüedad de los nombres, reducimos la dificultad y el tiempo que nos llevaría interpretar el código. No solo para nosotras, sino también para cualquier persona que pueda tener que trabajar con él en el futuro.

### Nombres reutilizados en el mismo *scope*

Aunque ahora tenemos el código en un estado mucho mejor, todavía tenemos un aspecto que no está realmente bien. La variable `$price` es actualizada constantemente y, no solo cambia de valor, sino que cambia de significado. En un momento dado, `$price` es el precio base del producto, en otro es el precio con el descuento aplicado y en otro es el precio con el impuesto aplicado.

```php
class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $price = $item->basePrice();
        $price = $this->applyDiscount($price);
        $price = $this->applyTax($price);
        return $price;
    }

    private function applyTax($price) {
        return $price * (1 + $this->tax);
    }

    private function applyDiscount($price) {
        return $price * (1 - $this->discount);
    }
}
```

El principal inconveniente de esta forma de programar es que tiene un coste de cambio alto en el futuro. Si hubiese que introducir algún paso nuevo en el cálculo del precio final, habría que modificar todos los lugares en los que se actualiza la variable `$price`. Además, el código es más difícil de comprender, ya que no es fácil saber en qué estado se encuentra `$price` en cada momento.

Podrías argumentar que se trata de una variable temporal y de que, en último término, lo que buscamos es el resultado final de todas las transformaciones. Por ejemplo, podríamos hacer algo así, y ni siquiera necesitaríamos la variable `$price`:

```php
class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $price = $this->applyTax(
            $this->applyDiscount(
                $item->basePrice()
            )
        );
        return $price;
    }

    private function applyTax($price) {
        return $price * (1 + $this->tax);
    }

    private function applyDiscount($price) {
        return $price * (1 - $this->discount);
    }
}
```

Bien, esta solución es interesante, pero a primera vista ya parece más difícil de leer. No solo eso, al igual que la solución inicial nos vamos a encontrar con problemas en caso de que en algún momento necesitemos introducir un cambio.

De entrada, sería más sencillo bautizar cada paso del cálculo con un nombre que refleje su significado. Por ejemplo:

```php
class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $basePrice = $item->basePrice();
        $discountedPrice = $this->applyDiscount($basePrice);
        $priceAfterTax = $this->applyTax($discountedPrice);
        return $priceAfterTax;
    }

    private function applyTax($price) {
        return $price * (1 + $this->tax);
    }

    private function applyDiscount($price) {
        return $price * (1 - $this->discount);
    }
}
```

## Más allá de los nombres: cuando aparecen conceptos

Por otro lado, este tipo de patrones en los que vamos cambiando el valor de una variable con cálculos que la toman como argumento de entrada, nos debería sugerir la presencia de un concepto que estaría mejor representado con un objeto. Como podemos ver, todas las operaciones se realizan sobre un precio, por lo que podríamos encapsularlo en un objeto `Price`. Y una vez que tenemos un objeto, lo más adecuado sería que éste se encargase de aplicar los descuentos y los impuestos.

```php
class Price {
    private $amount;

    public function __construct($amount) {
        $this->amount = $amount;
    }

    public function applyTax($tax) {
        $this->amount *= (1 + $tax);
    }

    public function applyDiscount($discount) {
        $this->amount *= (1 - $discount);
    }

    public function->amount() {
        return $this->amount;
    }
}

class Calculator {
    private $discount;
    private $tax;

    public function __construct($discount, $tax) {
        $this->discount = $discount;
        $this->tax = $tax;
    }

    public function finalPrice($item) {
        $price = new Price($item->basePrice());
        $price->applyDiscount($this->discount);
        $price->applyTax($this->tax);
        return $price->amount();
    }
}
```

Este refactor que acabamos de mostrar va más allá del alcance de este capítulo, pero nos ha servido para mostrar que cuando mejoramos los nombres, es muy fácil que afloren conceptos interesantes que no teníamos bien representandos con valores primitivos. En capítulos posteriores examinaremos cómo podemos introducir estos conceptos en nuestro código. 

### Tipo de palabra inadecuada

Los símbolos que, de algún modo, contradicen el concepto que representan son más difíciles de procesar, generalmente porque provocan una expectativa que no se cumple y, por tanto, debemos re-evaluar lo que estamos leyendo. Por ejemplo:

* Una acción debería representarse siempre mediante un verbo. 
* Un concepto, mediante un sustantivo.

A su vez, nunca nos sobran los adjetivos para precisar el significado del sustantivo, por lo que los nombres compuestos nos ayudan a representar con mayor precisión las cosas.

Volvamos al ejemplo. `Calculator` parece un buen nombre. `PriceCalculator` sería aún mejor, ya que hace explícito el hecho de que calcula precios. Es un sustantivo, por lo que se deduce que es un actor que hace algo. Veámosla como _interface_:

```php
interface PriceCalculator {
    public finalPrice(Item $item): float;
}
```
Obviamente, este refactor es un poco más arriesgado. Vamos a tocar una interfaz pública, pero también es verdad que con los IDE modernos este tipo de cambios es razonablemente seguro.

`finalPrice` es un sustantivo, pero en realidad ¿no representa una acción? ¿No sería mejor `calculateFinalPrice`?

```php
interface PriceCalculator {
    public calculateFinalPrice(Item $item): float;
}
```
Por un lado, es cierto que parece más imperativo. Estamos diciendo algo así como: "Calculadora: calcula el precio final". No deja lugar a dudas sobre lo que hace. En el lado negativo, resulta un nombre redundante, a la par que largo.

Pero antes… Volvamos un momento a la clase. `PriceCalculator`, ¿es un actor o una acción? A veces tendemos a ver los objetos como representaciones de objetos del _mundo real_. Sin embargo, podemos representar acciones y otros conceptos con objetos en el código. Esta forma de verlo puede cambiar por completo nuestra manera de hacer las cosas.

Supongamos entonces, que consideramos que `PriceCalculator` no es una _cosa_, sino una _acción_:

```php
interface CalculatePrice {
    public calculateFinalPrice(Product $product): float;
}
```

Tal y como está ahora, expresar ciertas cosas resulta extraño:

```php
$calculatePrice = new CalculatePrice();

$calculatePrice->calculateFinalPrice($product);
```

Pero podemos imaginarlo de otra forma mucho más fluida:

```php
$calculatePrice = new CalculatePrice();

$calculatePrice->finalForProduct($product);
```

Lo que nos deja con esta interfaz:

```php
interface CalculatePrice {
    public finalForProduct(Product $product): float;
}
```

Este cambio de nombre resulta interesante, pero también tenemos que valorarlo en su contexto. Que los objetos tengan nombres de acciones puede ser muy válido para representar casos de uso, pero no tanto para representar entidades de negocio. En este caso, si entendemos `CalculatePrice` como una entidad de negocio, `PriceCalculator` es un nombre mucho más adecuado.

### Números mágicos

A veces no se trata estrictamente de refactorizar nombres, sino de bautizar elementos que están presentes en nuestro código en forma de valores abstractos que tienen un valor de negocio que no ha sido hecho explícito.

Poniéndoles un nombre, lo hacemos. Lo que antes era:

```php
$vatAmount = $amountBeforeTaxes * .21;
```

En la línea anterior, `.21` es un número mágico. No sabemos qué significa, pero podemos intuir que se trata de un impuesto y deberíamos hacerlo explícito.

Después:

```php
const VAT_RATE = .21;
$vatAmount = $amountBeforeTaxes * VAT_RATE;
```

Convertir estos valores en constantes con nombre hace que su significado de negocio esté presente, sin tener que preocuparse de interpretarlo. Además, esto los hace reutilizables a lo largo de todo el código, lo que añade un plus de coherencia.

Así que, cada vez que encuentres uno de estos valores, hazte un favor y reemplázalo por una constante. Por ejemplo, los naturalmente ilegibles patrones de expresiones regulares:

```php
$isValidNif = preg_match('/^[0-9XYZ]\d{7}[^\dUIOÑ]$/', $nif);

// vs

$isValidNif = preg_match(VALID_NIF_PATTERN, $nif);
```

O los patrones de formato para todo tipo de mensajes:

```php
$mensaje = sprintf('¿Enviar un mensaje a %s en la dirección %s?', $user->username(), $user->email());

$mensaje = sprintf(CONFIRM_SEND_EMAIL_MESSAGE, $user->username(), $user->email());
```

## Nombres técnicos

Personalmente, me gustan poco los nombres técnicos formando parte de los nombres de variables, clases, interfaces, etc. De hecho, creo que en muchas ocasiones condicionan tanto el *naming*, que favorecen la creación de malos nombres.

Ya he hablado del problema de entender que los objetos en programación tienen que ser representaciones de objetos del mundo real. Esa forma de pensar nos lleva a ver todos los objetos como actores que hacen algo, cuando muchas veces son acciones.

En ocasiones, es verdad que tenemos que representar ciertas operaciones técnicas, que no todo va a ser negocio, pero eso no quiere decir que no hagamos las cosas de una manera elegante. Por ejemplo:

```php
interface BookTransformer
{
    public function transformToJson(Book $book): string;
    public function transformFromJson(string $bookDto): Book;
}

// vs

interface TransformBook
{
    public function toJson(Book $book): string;
    public function fromJson(string $bookDto): Book;
}
```

En cambio, en el dominio me choca ver cosas como:

```
class BookWasPrintedEvent implements DomainEvent
{
}

// vs

class BookWasPrinted implements DomainEvent
{
}
```

En este ejemplo, el uso del verbo en pasado debería ser suficiente para entender de un vistazo que está hablando de un evento, que no es otra cosa que un mensaje que indica que algo interesante ha ocurrido.

Es cierto que incluir algunos *apellidos técnicos* a nuestros nombres puede ayudarnos a localizar cosas en el IDE. Pero hay que recordar que no programamos para un IDE.

## Refactor de nombres

En general, gracias a las capacidades de refactor de los IDE o incluso del Buscar/Reemplazar en proyectos, realizar refactors de nombres es bastante seguro.

**Variables locales en métodos y funciones**. Cambiarlas no supone ningún problema, pues no afectan a nada que ocurra fuera de su ámbito.

**Propiedades y métodos privados en clases**. Tampoco suponen ningún problema al no afectar a nada externo a la clase.

**Interfaces públicas**. Aunque es más delicado, los IDE modernos deberían ayudarnos a realizarlos sin mayores problemas. La mayor dificultad me la he encontrado al cambiar nombres de clases, puesto que el IDE aunque localiza y cambia correctamente sus usos, no siempre identifica objetos relacionados, como los tests.

## El coste de un mal nombre

Imaginemos un sistema de gestión de bibliotecas que, inicialmente, se creó para gestionar libros. Simplificando muchísimo, aquí tenemos un concepto clave del negocio:

```php
class Book
{
    private $id;
    private $title;
    private $author;
    private $editor;
    private $year;
    private $city;
}
```

Con el tiempo la biblioteca pasó a gestionar revistas. Las revistas tienen número, pero tal vez en su momento se pensó que no sería necesario desarrollar una especialización:

```php
class Book
{
    private $id;
    private $title;
    private $author;
    private $editor;
    private $year;
    private $city;
    private $issue;
}
```

Y aquí comienza un desastre que solo se detecta mucho tiempo después y que puede suponer una sangría, quizá lenta pero constante, de tiempo, recursos y, en último término, dinero para los equipos y empresas.

La modificación de la clase `Book` hizo que esta pasara a representar dos conceptos distintos, pero quizá se consideró que era una ambigüedad manejable: un compromiso aceptable.

Claro que la biblioteca siguió evolucionando y con el avance tecnológico comenzó a introducir nuevos tipos de objetos, como CD, DVD, libros electrónicos, y un largo etcétera. En este punto, el conocimiento que maneja negocio y su representación en el código se han alejado tanto que el código se ha convertido en una pesadilla: ¿cómo sabemos si `Book` se refiere a un libro físico, a uno electrónico, a una película en DVD, a un juego en CD? Solo podemos saberlo examinando el contenido de cada objeto `Book`. Es decir: el código nos está obligando a pararnos a pensar para entenderlo. Necesitamos refactorizar y reescribir.

Es cierto que, dejando aparte el contenido, todos los objetos culturales conservados en una biblioteca comparten ese carácter de objeto cultural o soporte de contenidos. `CulturalObject` se nos antoja un nombre demasiado forzado, pero `Media` resulta bastante manejable:

```php
class Media
{
    private $id;
    private $signature;
    private $registeredSince;
    private $status;
}
```

`Media` representaría a los soportes de contenidos archivados en la biblioteca y que contendría propiedades como un número de registro (el _id_), la signatura topográfica (que nos comunica su ubicación física) y otros detalles relacionados con la actividad de archivo, préstamo, etcétera.

Pero esa clase tendría especializaciones que representan tipos de medios específicos, con sus propiedades y comportamientos propios.

```php
class Book extends Media
{
}

class Review extends Media
{
}

class ElectronicBook extends Media
{
}

class Movie extends Media
{
} 
```

Podríamos desarrollar más el conocimiento de negocio en el código, añadiendo interfaces. Por ejemplo, la gestión del préstamo:

```php
interface Lendable
{
    public function lend(User $user): void;
    public function return(DateTimeInterface $date): void;
}
```

Pero el resumen es que el hecho de no haber ido reflejando la evolución del conocimiento del negocio en el código nos lleva a tener un sobre-coste en forma de:

* El tiempo y recursos necesarios para actualizar el desarrollo a través de reescrituras.
* El tiempo y recursos necesarios para mantener el software cuando surgen problemas derivados de la mala representación del conocimiento.
* Las pérdidas por no ingresos debidos a la dificultad del software de adaptarse a las necesidades cambiantes del negocio.

Por esto, preocúpate por poner buenos nombres y mantenerlos al día. Va en ello tu salario.

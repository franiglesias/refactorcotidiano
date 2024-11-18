# Dónde poner el conocimiento

> En el que recurrimos a principios básicos de asignación de responsabilidades para averiguar qué objetos deberías saber qué cosas.

### Notas de la segunda edición

En este capítulo introduciremos los principios GRASP, los cuales habíamos mencionado en la edición anterior, por lo que cambiamos casi por completo los ejemplos.

## Refactorizar para trasladar conocimiento

Solemos decir que refactorizar tiene que ver con el **conocimiento** y el **significado**. Fundamentalmente, porque lo que hacemos es aportar significado al código con el objetivo de que este represente de una manera fiel y dinámica el conocimiento cambiante que tenemos del negocio del que nos ocupamos. Y, por otro lado, porque lo que perseguimos con un refactoring continuado es representar nuestro entendimiento actual del negocio en el código con la mayor precisión posible.

En el código de una aplicación tenemos objetos que representan alguna de estas cosas:

* **Conceptos**, ya sea en forma de entidades o de value objects. Las entidades representan conceptos que nos interesan por su identidad y tienen un ciclo de vida. Los value objects representan conceptos que nos interesan por su valor.
* **Relaciones entre esos conceptos**, que suelen representarse en forma de agregados y que están definidas por las reglas de negocio.
* **Procesos** que hacen interactuar los conceptos conforme a reglas de negocio también.

Uno de los problemas que tenemos que resolver al escribir código y al refactorizarlo es dónde poner el conocimiento y, más exactamente, las reglas de negocio.

Si hay algo que caracteriza al *legacy* es que el conocimiento sobre las reglas de negocio suele estar disperso a lo largo y ancho del código, en los lugares más imprevisibles y representado de las formas más dispares. El efecto de refactorizar este código es, esperamos, llegar a trasladar ese conocimiento al lugar donde mejor nos puede servir.

Pero incluso en código nuevo, el conocimiento puede estar disperso. Puede ser debido a que no conocemos bien nuestro negocio todavía, o porque no somos capaces de expresarlo mejor en un momento dado. Además, el código siempre va a tener un cierto desfase con lo que sabemos del negocio, porque el negocio cambia y nuestro conocimiento de él también.

Para saber donde colocar el conocimiento en el código podemos recurrir a varios principio y patrones.

## Principios básicos

**Principio de abstracción**. Benjamin Pierce formuló el principio de abstracción en su libro [*Types and programming languages*](https://www.amazon.es/Types-Programming-Languages-MIT-Press/dp/0262162091/ref=sr_1_1?adgrpid=56467442856&hvadid=275405870353&hvdev=c&hvlocphy=1005434&hvnetw=g&hvpos=1t1&hvqmt=e&hvrand=14620334178548921835&hvtargid=kwd-298006847943&keywords=types+and+programming+languages&qid=1555607597&s=gateway&sr=8-1):

>Each significant piece of functionality in a program should be implemented in just one place in the source code. Where similar functions are carried out by distinct pieces of code, it is generally beneficial to combine them into one by abstracting out the varying parts.

**DRY**. Por su parte, Andy Hunt y David Thomas, en [*The Pragmatic Programmer*](https://www.amazon.es/s?k=the+pragmatic+programmer&adgrpid=55802357883&hvadid=275519489680&hvdev=c&hvlocphy=1005434&hvnetw=g&hvpos=1t1&hvqmt=e&hvrand=18337966056430542312&hvtargid=kwd-302199567278&tag=hydes-21&ref=pd_sl_63qujbqsqo_e), presentan una versión de este mismo principio que posiblemente te sonará más: **Don't Repeat Yourself:**

>Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

En esencia, la idea que nos interesa recalcar es que cada regla de negocio estará representada en un único lugar y esa representación será la de referencia para todo el código.

Los principios que hemos enunciado se centran en el carácter único de la representación, pero no nos dicen dónde debe residir la misma. Lo cierto es que es un tema complejo, pues es algo que puede admitir varias interpretaciones y puede depender del estado de nuestro conocimiento actual del negocio.

## Buscando dónde guardar el conocimiento: patrones GRASP

Los patrones GRASP son un conjunto de patrones de diseño que nos ayudan a asignar responsabilidades a los objetos de un sistema. Fueron introducidos por Craig Larman, que identifica una serie de preguntas que nos podemos hacer para saber dónde colocar el conocimiento en un sistema orientado a objetos.

### Regla general: en los objetos que tienen la información necesaria

Este patrón se llama _Information Expert_ y es el más general de todos. Una responsabilidad se asignará al objeto que tenga la información necesaria para ejercerla.

En el contexto de refactoring, lo que nos dice este principio es que cuando estamos usando la información contenida en un objeto, ese uso o comportamiento tendría que estar en ese mismo objeto. Expresándolo en otras palabras, quiere decir que un objeto debe ser capaz de realizar todos los comportamientos que le sean propios, dentro del contexto de nuestra aplicación. Para ello no debería necesitar exponer sus propiedades internas o estado.

Por tanto, cuando preguntamos a un objeto sobre su estado y realizamos acciones basadas en la respuesta, lo suyo debería ser encapsular esas acciones en forma de comportamientos del objeto. Para ello, podemos seguir el principio *Tell, don't ask*. Esto es, en lugar de obtener información de un objeto para operar con ella y tomar una decisión sobre ese objeto, le pedimos que lo haga él mismo y nos entregue un resultado si es adecuado para el contexto. Esto lo trataremos con más detalle en el siguiente capítulo.

Los *value objects* y *entidades* son lugares ideales para encapsular conocimiento de dominio.

Supongamos que en nuestro negocio estamos interesados en ofrecer ciertos productos o ventajas a usuarios cuya cuenta de correo pertenezca a ciertos dominios. Un ejemplo de esto son los programas de beneficios de algunas empresas. El correo electrónico es, pues, un concepto importante del negocio y lo representamos mediante un _value object_:

```php
class Email
{
    private string $email;

    public function __construct(string $email)
    {
        $this->email = $email;
    }
    
    public static function valid(string $email)
    {
        if (! filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException(sprintf('%s is not valid email.', $email));
        }
        return new self($email)
    }

    public function __toString(): string
    {
        return $this->$email;
    }
}
```

En un momento dado nos puede interesar saber si un empleado tiene acceso a un beneficio concreto, cosa que vamos a controlar obteniendo la lista de dominios corporativos que lo ofrecen. Podríamos hacerlo de esta manera:

```php
class CanSeeBenefit
{
    public function byCorporateEmail(Email $email)
    {
        [, $domain] = explode('@', (string)$email);

        if (!in_array($domain, $this->getBenefitDomains->execute(), true)) {
            return false;
        }
        
        return true
    }
}
```

Como se puede ver, estamos pidiendo su valor a `$email` para poder extraer el dominio y compararlo con la lista de dominios. Por definición, sabemos que un email se compone de un nombre de usuario y un dominio, así que lo lógico sería preguntarle a `$email` por su dominio y no calcularlo fuera de él.

```php
class Email
{
    private string $email;

    public function __construct(string $email)
    {
        $this->email = $email;
    }
    
    public static function valid(string $email)
    {
        if (! filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException(sprintf('%s is not valid email.', $email));
        }
        return new self($email)
    }

    public function domain(): string
    {
        [, $domain] = explode('@', $this->email);

        return $domain;
    }

    public function __toString(): string
    {
        return $this->$email;
    }
}
```

Este es un primer paso para trasladar el conocimiento al lugar donde mejor se puede usar.

```php
class CanSeeBenefit
{
    public function byCorporateEmail(Email $email)
    {
        if (!in_array($email->domain(), $this->getBenefitDomains->execute(), true)) {
            return false;
        }
        
        return true
    }
}
```

Pero en el fondo esto no soluciona completamente el problema. Ahora podemos obtener el dominio de un email y, aunque se obtenga de un cálculo, no deja de ser el acceso a una propiedad. Cierto es que lo hemos implementado de tal forma que necesitamos calcular el dominio, pero podría no ser así. Mira, por ejemplo, esta versión:

```php
class Email
{
    private string $username;
    private string $domain;

    private function __construct(string $username, string $domain)
    {
        $this->username = $username;
        $this->domain = $domain;
    }

    public static function valid(string $email): self
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException(sprintf('%s is not a valid email.', $email));
        }

        [$username, $domain] = explode('@', $email);
        return new self($username, $domain);
    }

    public function domain(): string
    {
        return $this->domain;
    }

    public function __toString(): string
    {
        return $this->username . '@' . $this->domain;
    }
}
```

Entonces, ¿qué podríamos hacer? Pues la respuesta es invertir la cuestión, En lugar de extraer si el dominio del Email para mirar si está en la lista que tenemos, lo suyo es pasarle la lista para que nos diga si su dominio está en ella:

```php
class Email
{
    private string $username;
    private string $domain;

    private function __construct(string $username, string $domain)
    {
        $this->username = $username;
        $this->domain = $domain;
    }

    public static function valid(string $email): self
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException(sprintf('%s is not a valid email.', $email));
        }

        [$username, $domain] = explode('@', $email);
        return new self($username, $domain);
    }

    public function belongsToOneOfThisDomains(array $domains): bool
    {
        return in_array($this->domain, $domains, true);
    }

    public function __toString(): string
    {
        return $this->username . '@' . $this->domain;
    }
}
```

Ahora podemos usarlo así:

```php
class CanSeeBenefit
{
    public function byCorporateEmail(Email $email)
    {
        return $email->belongsToOneOfThisDomains($this->getBenefitDomains->execute());
    }
}
```

Con este cambio resulta que `Email` ya no tiene que mostrar ninguna de sus propiedades. Esto nos da libertad para cambiar su implementación sin tener que cambiar el código que lo usa. Y, por otro lado, expone comportamiento que puede ser usado por otros objetos interesados.

### Quien se ha de encargar de construir un objeto

Este patrón se llama _Creator_ y nos dice que la responsabilidad de crear un objeto debe recaer en aquel que tenga la información necesaria para hacerlo, o bien que coleccione o agrupe los objetos que se van a crear.

El ejemplo paradigmático de este patrón es el de un objeto factura y sus líneas. La responsabilidad de crear una línea de factura debería recaer en la propia factura, ya que aunque no tenga la información necesaria para crearla, sí que agrupa las líneas de factura. De hecho, las líneas de factura no tienen sentido fuera de una factura.  

```php
class Invoice
{
    private array $lines = [];

    public function addLine(string $description, float $amount): void
    {
        $this->lines[] = new InvoiceLine($description, $amount);
    }
}

class InvoiceLine
{
    private string $description;
    private float $amount;

    public function __construct(string $description, float $amount)
    {
        $this->description = $description;
        $this->amount = $amount;
    }
}
```

### Reglas de negocio como _Specification_

El ejemplo anterior es una primera aproximación a cómo mover el conocimiento. En este caso no se trata tanto de la regla de negocio como de un requisito para poder implementarla.

Podríamos decir que la regla de negocio implica distintos conocimientos. En términos de negocio nuestro ejemplo se enunciaría como "todos los clientes cuyo dominio de correo esté incluido en la lista tienen derecho a tal ventaja cuando realicen tal acción". Técnicamente, implica saber sobre usuarios y sus emails, y saber extraer su dominio de correo para saber si está incluido en tal lista.

Desde el punto de vista del negocio la regla relaciona clientes, seleccionados por una característica, con una ventaja que les vamos a otorgar.

Ese conocimiento se puede encapsular en una **Specification**, que no es más que un objeto que puede decidir si otro objeto cumple una serie de condiciones. El objeto en que estamos interesadas es `Customer` y para el contexto de esta regla, nos interesa poder preguntarle por su dominio de correo.

```php
<?php
declare(strict_types=1);

namespace App\Domain;

class BelongsToDomainEligibleForPromotion
{
    public $domains = [
        'example.com',
        'example.org'
    ];
    
    public function isSatisfiedBy(Customer $customer): bool
    {
        return $this->isEmailEligibleForPromotion($customer->email());
    }
    
    private function isEmailEligibleForPromotion($domain) {
        if (in_array($email->domain(), $this->domains, true)) {
            return true;
        }
        
        return false;
    }
}
```

Ahora el conocimiento de la regla de negocio se encuentra en un solo lugar y lo puedes reutilizar allí donde lo necesites[^fn-spec]

[^fn-spec]: Una objeción que se puede poner a este código es que instanciamos la _Specification_. Normalmente, lo mejor sería inyectar en el servicio una factoría de _Specification_ para pedirle las que necesitemos y que sea la factoría la que gestione sus posibles dependencias.

```php
<?php
declare(strict_types=1);

namespace App\Domain;

class OfferPromotion
{
    public function applyTo(Order $order)
    {
        $eligibleForPromotion = new BelongsToDomainEligibleForPromotion();
        
        if ($eligibleForPromotion->isSatisfiedBy($order->customer())) {
            $order->applyPromotion($this);
        }
    }
}
```

No solo eso, sino que incluso nos permite escribir mejor el servicio al expresar las relaciones correctas: en este caso la regla de negocio se basa en una propiedad de los clientes y no de los pedidos, aunque luego se aplique el resultado a los pedidos o al cálculo de su importe.

Sobre el patrón _Specification_ puedes encontrar [más información en este artículo](https://franiglesias.github.io/patron-specification-del-dominio-a-la-infraestructura-1/)



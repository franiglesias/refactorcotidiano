# Aplica Tell, Don't Ask

> En el que buscamos empujar el comportamiento dentro de nuestros objetos para que sepan hacer cosas por sí mismos, en lugar de preguntarles por lo que saben. 

Hasta ahora, hemos trabajado refactors muy orientados a mejorar la expresividad del código y a la organización de unidades de código. Pero en estos capítulos nos enfocaremos en los principios de diseño orientado a objetos.

Los principios de diseño nos proporcionan criterios útiles tanto para guiarnos en el desarrollo como para evaluar código existente en el que tenemos que intervenir.

### Notas de la segunda edición

En esta revisión, hemos dividido este capítulo en dos a fin de tratar de forma más detallada los principios presentados: _Tell, Don't Ask_ y la _Ley de Demeter_. Esto nos permitirá explorarlos con más detalle y mejores ejemplos.

## Tell, don't ask

La traducción de este enunciado a español sería algo así como "Pide, no preguntes". La idea de fondo de este principio es que cuando queremos modificar un objeto basándose su propio estado, no es buena idea preguntarle por su estado (*ask*), hacer el cálculo y cambiar su estado si fuera preciso. En su lugar, lo propio sería encapsular ese proceso en un método del propio objeto y decirle (*tell*) que lo realice él mismo.

Dicho en otras palabras: cada objeto es responsable de su estado representado por sus propiedades internas, y lo mantiene oculto a los demás objetos, que solo conocerán su interfaz pública. Este principio se conoce como _Information hiding_. Consecuentemente, las relaciones entre los objetos deben producirse siempre mediante llamadas a métodos, evitando el acceso directo a las propiedades internas de otros objetos.

Supongamos que tenemos una clase `Square` que representa un cuadrado y queremos poder calcular su área:

```php

class Square
{
    private $side;

    public function __construct($side)
    {
        $this->side = $side;
    }

    public function side()
    {
        return $this->side;
    }
}
```

Aquí tenemos una posible implementación de un `AreaCalculator`.

```

class AreaCalculator
{
    public function calculate($square)
    {
        $side = $square->side();
        return $side**2;
    }
}
```

Ahora, imaginemos que queremos calcular el área de otras figuras geométricas, como el triángulo o el círculo:

```php
class Triangle
{
    private $base;
    private $height;

    public function __construct($base, $height)
    {
        $this->base = $base;
        $this->height = $height;
    }

    public function base()
    {
        return $this->base;
    }

    public function height()
    {
        return $this->height;
    }
}
```

¿Cómo le explicamos a `AreaCalculator` que ahora tiene que calcular el área de un triángulo? Podríamos fijarnos en su tipo, para decidir qué algoritmo se debe aplicar:

```php
class AreaCalculator
{
    public function calculate($shape)
    {
        if ($shape instanceof Square) {
            $side = $shape->side();
            return $side**2;
        }

        if ($shape instanceof Triangle) {
            $base = $shape->base();
            $height = $shape->height();
            return ($base * $height) / 2;
        }
    }
}
```

Como se puede ver en el código, `AreaCalculator` tiene que saber un montón de cosas acerca de los objetos de lo que tiene que calcular su área:

* Necesita saber qué tipo de objeto es (`Square` o `Triangle`).
* Necesita saber qué propiedades tiene cada objeto, según el tipo de objeto (`side` o `base` y `height`).
* Necesita saber cómo calcular el área de cada objeto.

Esto viola todos los principios relacionados con la encapsulación. `AreaCalculator` tiene que conocer demasiados detalles de los objetos que tiene que calcular. Además, si añadimos una nueva figura geométrica, tendremos que modificar `AreaCalculator` para añadir una nueva decisión:

```php
class Circle
{
    private $radius;

    public function __construct($radius)
    {
        $this->radius = $radius;
    }

    public function radius()
    {
        return $this->radius;
    }
}
```

Y una nueva modificación en `AreaCalculator`:

```php
class AreaCalculator
{
    public function calculate($shape)
    {
        if ($shape instanceof Square) {
            $side = $shape->side();
            return $side**2;
        }

        if ($shape instanceof Triangle) {
            $base = $shape->base();
            $height = $shape->height();
            return ($base * $height) / 2;
        }

        if ($shape instanceof Circle) {
            $radius = $shape->radius();
            return pi() * $radius**2;
        }
    }
}
```

En este tipo de situaciones es donde el principio *Tell, Don't Ask* nos puede ayudar. En lugar de preguntar a los objetos por su estado y calcular el área en otro lugar, podemos pedirles que lo hagan ellos mismos. El área es una propiedad de las figuras geométricas y cada una de ellas tiene acceso a los datos necesarios para calcularla. Por tanto, el cálculo del área debería estar en la clase de cada figura.

```php
class Square
{
    private $side;

    public function __construct($side)
    {
        $this->side = $side;
    }

    public function side()
    {
        return $this->side;
    }

    public function area()
    {
        return $this->side**2;
    }
}
```
```php
class Triangle
{
    private $base;
    private $height;

    public function __construct($base, $height)
    {
        $this->base = $base;
        $this->height = $height;
    }

    public function base()
    {
        return $this->base;
    }

    public function height()
    {
        return $this->height;
    }

    public function area()
    {
        return ($this->base * $this->height) / 2;
    }
}
```
```php
class Circle
{
    private $radius;

    public function __construct($radius)
    {
        $this->radius = $radius;
    }

    public function radius()
    {
        return $this->radius;
    }

    public function area()
    {
        return pi() * $this->radius**2;
    }
}
```

Puesto que ahora cada figura sabe cómo calcular su área, `AreaCalculator` se simplifica. Vamos a hacerlo por pasos. En el primero nos limitamos a reemplazar el cálculo del área por el método `area` de cada figura. Así quedan tras la introducción del principio *Tell, Don't Ask* a nuestro problema.

```php
class AreaCalculator
{
    public function calculate($shape)
    {
        if ($shape instanceof Square) {
            return $shape->area();
        }
        
        if ($shape instanceof Triangle) {
            return $shape->area();
        }
        
        if ($shape instanceof Circle) {
            return $shape->area();
        }
    }
}
```

## Polimorfismo

Bien, esto tiene otra pinta. Sin embargo, seguimos preguntándole cosas a las figuras geométricas. En este caso, preguntamos si son instancias de `Square`, `Triangle` o `Circle`, y es obvio que la sucesión de `if` es redundante porque en último término les pedimos que hagan lo mismo. Dado que cada figura expone un método `area`, podemos asumir que todas ellas son capaces de responder al mismo mensaje. Por tanto, simplifiquemos `AreaCalculator`:

```php
class AreaCalculator
{
    public function calculate($shape)
    {
        return $shape->area();
    }
}
```
Y aquí podemos ver el principio *Tell, Don't Ask* en acción. En lugar de preguntar a las figuras geométricas por su estado y calcular el área en otro lugar, simplemente les pedimos que lo hagan ellas mismas independientemente de su tipo.

Esta solución es posible gracias al _polimorfismo_. El polimorfismo es una característica de los lenguajes orientados a objetos gracias a la cual podemos enviar el mismo mensaje a diferentes objetos, obteniendo respuestas basándonos en su tipo específico. Esto nos permite evitar tener que preguntar por el tipo de objeto cada vez y actuar en consecuencia.

El tipo muchas veces viene dado por la clase de los objetos, que es lo que ocurre en nuestro ejemplo, pero en otros casos es una propiedad de una única clase. El problema, en nuestro caso, es que al principio teníamos objetos anémicos a los que no podíamos enviar ningún mensaje. Una vez que hemos aplicado _Tell, don't ask_ y hemos movido el comportamiento a los objetos, nos hemos dado cuenta de que podríamos beneficiarnos del polimorfismo.

Por otro lado, nos hemos aprovechado de la posibilidad de PHP, y de otros lenguajes, de hacer "duck typing", gracias a lo cual el método `calculate` no requiere tipado. En lugar de preguntar por el tipo de objeto, simplemente le pedimos que haga algo. Si el objeto sabe hacerlo, lo hará. Si no, lanzará una excepción. En algunos lenguajes de programación tendríamos que haber declarado el tipo de `$shape` en el método `calculate` para saber que se puede llamar a `area`.

Examinando las regularidades de las clases `Square`, `Triangle` y `Circle`, podemos ver que todas ellas tienen un método `area` que devuelve un valor numérico. Por tanto, podemos definir una interfaz común para todas ellas:

```php
interface Shape
{
    public function area(): float;
}
```

Gracias a esto, `AreaCalculator` puede confiar en que cualquier objeto que implemente la interfaz `Shape` tendrá un método `area` que devolverá un valor numérico. En caso de que el objeto recibido no implemente la interfaz `Shape`, PHP lanzará una excepción antes incluso de intentar ejecutar el método `calculate`.

```php
class AreaCalculator
{
    public function calculate(Shape $shape): float
    {
        return $shape->area();
    }
}
```

Una ventaja extra, como se puede ver, es que ahora `AreaCalculator` no tiene que preocuparse por los detalles de cada figura geométrica. Solo necesita saber que el objeto que recibe implementa la interfaz `Shape` y que, por tanto, tiene un método `area` que puede llamar. Eso hace innecesario acceder directamente a las propiedades internas de cada figura.

```php
class Square implements Shape
{
    private $side;

    public function __construct($side)
    {
        $this->side = $side;
    }

    public function area(): float
    {
        return $this->side**2;
    }
}
```

```php
class Triangle implements Shape
{
    private $base;
    private $height;

    public function __construct($base, $height)
    {
        $this->base = $base;
        $this->height = $height;
    }

    public function area(): float
    {
        return ($this->base * $this->height) / 2;
    }
}
```

```php
class Circle implements Shape
{
    private $radius;

    public function __construct($radius)
    {
        $this->radius = $radius;
    }

    public function area(): float
    {
        return pi() * $this->radius**2;
    }
}
```

## Resumen del capítulo


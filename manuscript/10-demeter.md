# Aplica la Ley de Demeter

> En el que seguimos hablando acerca de la redistribución de responsabilidades. Un sistema orientado a objetos se basa en los mensajes que se envían los objetos entre sí, por lo que resulta importante aprender qué objetos deberían hablar entre sí y cuáles no. 

## Ley de Demeter

La **Ley de Demeter**[^fn-demeter], que también se conoce como **Principio de mínimo conocimiento**, dice que un objeto no debería conocer la organización interna de los otros objetos con los que colabora. Lo único que debería saber es cómo comunicarse con ellos. De este modo, al conocer lo mínimo posible, se acopla mínimamente a ellos.

[^fn-demeter]: El nombre viene del proyecto donde se usó por primera vez.

### Notas de la segunda edición

Como mencionamos en el capítulo anterior, hemos separado los capítulos de _Tell, Don't Ask_ y _Ley de Demeter_ en dos capítulos distintos. Aunque ambos tratan sobre la redistribución de responsabilidades, el anterior se centra en la encapsulación de la lógica de negocio, mientras que este se centra en la comunicación entre objetos.

### Cumpliendo la Ley

Siguiendo la **Ley de Demeter**, como veremos, un método de una clase solo puede hablar con los objetos a los que conoce. Estos son:

* La propia clase, de la que puede usar todos sus métodos.
* Objetos que son propiedades de esa clase.
* Objetos creados en el mismo método que los usa.
* Objetos pasados como parámetros a ese método.

La finalidad de la **ley de Demeter** es evitar el acoplamiento estrecho entre objetos. Si un método usa un objeto, contenido en otro objeto que ha recibido o creado, implica un conocimiento que va más allá de la interfaz pública del objeto intermedio.

Vamos a ver esto con un ejemplo un poco amañado, pero que nos permitirá ilustrar varios problemas, empezando por algunos más superficiales y llegando a otros relacionados con el diseño de la solución.

Aquí tenemos una clase `Product` que representa un producto en una tienda online. Cada producto tiene un precio unitario y una promoción que se aplica si se compran más unidades de una cantidad determinada.

```php
class Product
{
    private float $unitPrice;
    private Promotion $promotion;

    public function __construct(float $unitPrice, Promotion $promotion)
    {
        $this->unitPrice = $unitPrice;
        $this->promotion = $promotion;
    }

    public function unitPrice(): float
    {
        return $this->unitPrice;
    }

    public function currentPromotion(): Promotion
    {
        return $this->promotion;
    }
}
```

Aquí tenemos Promotion, que contiene la información sobre el descuento aplicable.

```php
class Promotion
{
    private int $threshold;
    private float $discountRate;

    public function __construct(int $threshold, float $discountRate)
    {
        $this->threshold = $threshold;
        $this->discountRate = $discountRate;
    }

    public function threshold(): int
    {
        return $this->threshold;
    }

    public function discountRate(): float
    {
        return $this->discountRate;
    }
}
```

Esta es la calculadora de precios. Aquí podemos ver que necesitamos obtener la Promotion con el fin de obtener los descuentos aplicables y el límite mínimo de unidades.

```php
class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        $promotion = $product->currentPromotion();
        if ($units > $promotion->threshold()) {
            return $product->unitPrice() * $units * (1 - $promotion->discountPct());
        }
        return $product->unitPrice() * $units;
    }
}
```

En el ejemplo, el método `calculatePrice` obtiene el descuento aplicable llamando a un método de `Product`, que devuelve otro objeto al cual le preguntamos sobre los datos necesarios para realizar el descuento.

La violación de la Ley de Demeter se produce aquí porque el método `calculatePrice` está hablando con un objeto que no conoce directamente, sino a través de otro. ¿Qué objeto es este y cuál es su interfaz? Podemos suponer que se trata de un objeto `Promotion`, pero eso es algo que sabemos nosotros, no el código. Este es el exceso de conocimiento que la Ley de Demeter trata de evitar.

¿Cómo podemos refactorizar este tipo de problemas y cumplir con la Ley de Demeter? Antes de nada, hay que advertir que no existe una solución única. Esta dependerá del contexto y de la correcta atribución de responsabilidades a los distintos objetos.

### Pasa el objeto intermedio a otro método del objeto consumidor

Aplicando la letra de la Ley de Demeter, un método que recibe el objeto intermedio como parámetro no viola la ley. Lo ideal es que sea el único parámetro del método, cosa que no siempre es fácil de conseguir. Por ejemplo, podríamos hacer algo así, devolviendo el porcentaje de descuento o cero si no se puede aplicar:

```php 
class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        $discountPct = $this->discountPct($product->currentPromotion(), $units);
        
        return $product->unitPrice() * $units * (1 - $discountPct);
    }

    private function discountPct(Promotion $promotion): float
    {
        if ($units > $promotion->threshold()) {
            return $promotion->discountRate();
        }
        return 0;
    }
}
```

### Encapsular el comportamiento en el objeto que conocemos (Tell, Don't Ask)

En algunos casos, en lugar de pedirle a nuestro objeto conocido que nos dé el otro objeto, podemos pedirle que haga lo que sea necesario con él y nos devuelva el resultado. En este caso, podríamos pedirle a `Product` que nos devuelva el precio total para un número de unidades. De esta forma, ni siquiera necesitaríamos saber si hay una promoción o no.

```php
class Product
{
    public function totalPrice(int $units): float
    {
        $promotion = $this->currentPromotion();
        if ($units > $promotion->threshold()) {
            return $this->unitPrice() * $units * (1 - $promotion->discountRate());
        }
        return $this->unitPrice() * $units;
    }
}

class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        return $product->totalPrice($units);
    }
}
```

Esta es una solución interesante para muchos casos. Si tenemos modelos anémicos, tendemos a usar sus propiedades de forma directa, o a través de getters, para hacer cálculos. En lugar de eso, podemos pedirle al objeto que haga el cálculo por nosotros, ya que contiene toda la información necesaria y, por tanto, debería ser su responsabilidad.

Esto es una clara mejora con respecto a la situación inicial, pero seguramente podemos hacerlo mejor. Hay un par de cosas que nos chirrían:

* ¿Por qué no aplicar el mismo principio Tell, Don't Ask a `Promotion`? Estamos preguntando por el descuento y el umbral, cuando podríamos pedirle a `Promotion` que nos devuelva el descuento aplicable para el número de unidades.
* ¿Tiene sentido que Product sepa cómo calcular el precio total? Es más, ¿tiene sentido que Product sepa si hay una promoción o no? ¿No sería mejor que alguien más se encargara de eso?

### Moviendo responsabilidades: otra vez Tell, Don't Ask

Retomemos el primer intento de refactor. En ese caso, pasábamos el objeto intermedio como parámetro a otro método, siguiendo la letra de la ley de Demeter. Igualmente, podemos ver que estamos preguntando a `Promotion` por el descuento y el umbral, cuando podríamos pedirle que nos devuelva el descuento aplicable para un número de unidades.

```php 
class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        $discountPct = $this->discountPct($product->currentPromotion(), $units);
        
        return $product->unitPrice() * $units * (1 - $discountPct);
    }

    private function discountPct(Promotion $promotion, int $units): float
    {
        if ($units > $promotion->threshold()) {
            return $promotion->discountRate();
        }
        return 0;
    }
}
```

Es decir, dado que `Promotion` es el que sabe cómo calcular el descuento aplicable, es lógico que sea él quien lo haga. Y lo único que tenemos que hacer es encapsular esa lógica en un método de `Promotion`.

```php
class Promotion
{
    public function discountPct(int $units): float
    {
        if ($units > $this->threshold()) {
            return $this->discountRate();
        }
        return 0;
    }
}

class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        $discountPct = $this->discountPct($product->currentPromotion(), $units);
        
        return $product->unitPrice() * $units * (1 - $discountPct);
    }

    private function discountPct(Promotion $promotion, int $units): float
    {
        return $promotion->discountPct($units);
    }
}
```

Pero, como hemos dicho, ya que `Promotion` está contenido en `Product`, lo suyo sería que pedirle a Product que se encargue de calcular el descuento. De esta forma, `Product` sería el que conoce la estructura de precios y cómo aplicar los descuentos y no tendríamos que saber nada de `Promotion` fuera de `Product`.

```php
class Promotion
{
    public function discountPct(int $units): float
    {
        if ($units > $this->threshold()) {
            return $this->discountRate();
        }
        return 0;
    }
}

class Product
{
    public function totalPrice(int $units): float
    {
        $promotion = $this->currentPromotion();
        return $this->unitPrice() * $units * (1 - $promotion->discountPct());
    }
}

class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        return $product->totalPrice($units);
    }
}
```

Como nos preguntábamos antes, nos molesta un poco que `Product` sepa tanto sobre precios. `Promotion` podría ser mejor lugar para tener la lógica de descuentos. ¿Y si `Promotion` fuese la encargada de calcular el precio total?

```php
class Promotion
{
    public function totalPrice(float $unitPrice, int $units): float
    {
        if ($units > $this->threshold()) {
            return $unitPrice * $units * (1 - $this->discountRate());
        }
        return $unitPrice * $units;
    }
}

class Product
{
    public function totalPrice(int $units): float
    {
        $promotion = $this->currentPromotion();
        return $promotion->totalPrice($this->unitPrice(), $units);
    }
}

class PriceCalculator
{
    public function calculatePrice(Product $product, int $units): float
    {
        return $product->totalPrice($units);
    }
}
```

### Reasignación de responsabilidades

Si lo piensas cuidadosamente, esta nueva distribución de responsabilidades tiene mucho sentido. `Promotion` está encapsulando una estrategia de descuentos, mientras que `Product` solo tiene que preocuparse de su precio unitario y de aplicar la promoción. Esto parece vacíar de contenido a `PriceCalculator`, por lo que podríamos eliminarlo. Sin embargo, lo habitual sería que una tienda on-line pueda tener varios tipos de promociones y estrategias de precios. Sigue teniendo sentido que haya un objeto que se encargue de calcular el precio total de un producto, pero lo que no está bien es que una promoción está tan estrechamente ligada a un producto. Querremos poder tener la flexibilidad de gestionar los productos, por un lado, y las promociones y estrategias de precios, por otro.

Esto nos debería sonar a un patrón _Strategy_ para decirle a `PriceCalculator` qué estrategia de precios aplicar al producto que le pasamos. Por tanto, en lugar de que `Price` contenga una referencia a `Promotion`, lo que hacemos es separarlos:

```php 
class Promotion
{
    private float $discountRate;
    private int $threshold;
    
    public function totalPrice(float $unitPrice, int $units): float
    {
        if ($units > $this->threshold) {
            return $unitPrice * $units * (1 - $this->discountRate);
        }
        return $unitPrice * $units;
    }
}

class Product
{
    private float $unitPrice;
    
    public function totalPrice(int $units, Promotion $promotion): float
    {
        return $promotion->totalPrice($this->unitPrice, $units);
    }
}

class PriceCalculator
{
    public function calculatePrice(Product $product, int $units, Promotion $promotion): float
    {
        return $price->totalPrice($units, $promotion);
    }
}
```

Aprovecho para llamar tu atención sobre la clase Producto. A fin de no exponer sus propiedades, sino de encapsularlas, lo que hacemos es pasarle el objeto `Promotion` y que sea él quien se encargue de calcular el precio total, para lo cual `Product` le pasa la información de su precio unitario.

```php
class Product
{
    private float $unitPrice;
    
    public function totalPrice(int $units, Promotion $promotion): float
    {
        return $promotion->totalPrice($this->unitPrice, $units);
    }
}
```

## Resumen del capítulo

La Ley de Demeter o Principio de Mínimo Conocimiento nos pide que un objeto no debería conocer la estructura interna de los objetos con los que colabora. Solo tiene permitido hablar con aquellos objetos a los que puede conocer directamente, los cuales son:

* La propia clase, de la que puede usar todos sus métodos.
* Objetos que son propiedades de esa clase.
* Objetos creados en el mismo método que los usa.
* Objetos pasados como parámetros a ese método.

Aplicando este principio, podemos mejorar la organización de nuestro código distribuyendo las responsabilidades entre los objetos participantes.
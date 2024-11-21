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


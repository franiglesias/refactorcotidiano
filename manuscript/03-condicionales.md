# Capítulo 3. Acondiciona las condicionales

> En el que decidimos cómo hacer que las decisiones que el código toma sean más comprensibles y fáciles de mantener en el futuro, porque al fin y a la postre todo en esta vida es decidir. El caso es saber cuando tomar las decisiones y comprender bien sus condiciones y sus consecuencias.

### Notas de la segunda edición

En este capítulo, como en otros, hemos mejorado los ejemplos y corregido algunas explicaciones para que sean más claras. Ahora la mayor parte de ejemplos tienen más sentido y ofrecen más contexto. Asímismo hemos incluido algunas propuestas nuevas.

## La complejidad de decidir

Es bastante obvio que si hay algo que añade complejidad a un software es la **toma de decisiones** y, por tanto, las estructuras condicionales con las que la expresamos.

Estas estructuras pueden introducir dificultades de comprensión debido a varias razones:

* **La complejidad de las expresiones evaluadas**, sobre todo cuando se combinan mediante operadores lógicos tres o más condiciones, lo que las hace difíciles de procesar para nosotras.
* **La anidación de estructuras condicionales** y la concatenación de condicionales mediante `else`, introduciendo múltiples flujos de ejecución.
* **El desequilibrio entre las ramas** en las que una rama tiene unas pocas líneas frente a la otra que esconde su propia complejidad, lo que puede llevar a pasar por alto la rama corta y dificulta la lectura de la rama larga al introducir un nuevo nivel de indentación.

## ¿Cuándo refactorizar condicionales?

En general, como regla práctica, hay que refactorizar condicionales cuando su lectura no nos deja claro cuál es su significado. Esto se aplica en las dos partes de la estructura:

* **La expresión condicional**: qué define lo que tiene que pasar para que el flujo se dirija por una o por otra rama.
* **Las ramas**: las diferentes acciones que se deben ejecutar en caso de cumplirse o no la condición.

También podemos aplicar alguna de las reglas de _object calisthenics_:

**Aplanar niveles de indentación**: cuanto menos anidamiento en el código, más fácil de leer es porque indica que no estamos mezclando niveles de abstracción. El objetivo sería tener un solo nivel de indentación en cada método o función.

**Eliminar `else`**: en muchos casos, es posible eliminar ramas alternativas, bien directamente, bien encapsulando toda la estructura en un método o función.

Vamos a ver como podemos proceder a refactorizar condicionales de una forma sistemática.

## La rama corta primero

Si una estructura condicional nos lleva por una rama muy corta en caso de cumplirse y por una muy larga en el caso contrario, se recomienda que la rama corta sea la primera, para evitar que pase desapercibida. Por ejemplo, este fragmento tan feo:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod === null) {
        $logger = Logger::getInstance();
        $logger->debug("Medio de pago desconocido");
        if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
            $paymentMethod = new PaypalPaymentMethod();
        } else {
            $paymentMethod = new DefaultPaymentMethod();
        }
    } else {
        $paymentMethod = $selectedPaymentMethod;
    }
    
    return $paymentMethod;
}
```

Podría reescribirse así y ahora es más fácil ver la rama corta. Esto nos va a encaminar hacia otras refactorizaciones que veremos más adelante:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        $paymentMethod = $selectedPaymentMethod;
    } else {
        $logger = Logger::getInstance();
        $logger->debug("Medio de pago desconocido");
        if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
            $paymentMethod = new PaypalPaymentMethod();
        } else {
            $paymentMethod = new DefaultPaymentMethod();
        }
    }
    
    return $paymentMethod;
}
```

## Return early

Si estamos dentro de una función o método y podemos hacer el retorno desde dentro de una rama es preferible hacerlo. Con eso podemos evitar la cláusula `else` y hacer que el código vuelva al nivel de indentación anterior, lo que facilitará la lectura. Veámoslo aplicado al ejemplo anterior:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        $paymentMethod = $selectedPaymentMethod;
        
        return $paymentMethod;
    } else {
        $logger = Logger::getInstance();
        $logger->debug("Medio de pago desconocido");
        if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
            $paymentMethod = new PaypalPaymentMethod();
        } else {
            $paymentMethod = new DefaultPaymentMethod();
        }
    }
    
    return $paymentMethod;
}
```

No hace falta mantener la variable temporal, pues podemos devolver directamente la respuesta obtenida:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    } else {
        $logger = Logger::getInstance();
        $logger->debug("Medio de pago desconocido");
        if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
            $paymentMethod = new PaypalPaymentMethod();
        } else {
            $paymentMethod = new DefaultPaymentMethod();
        }
        
        return $paymentMethod;
    }
}
```

Y, por último, podemos eliminar la cláusula `else` y dejarlo así.

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    }
    
    $logger = Logger::getInstance();
    $logger->debug("Medio de pago desconocido");
    if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
        $paymentMethod = new PaypalPaymentMethod();
    } else {
        $paymentMethod = new DefaultPaymentMethod();
    }
    
    return $paymentMethod;
}
```

Aplicando el mismo principio, reducimos la complejidad aportada por la condicional final, retornando directamente y eliminando tanto `else` como las variables temporales:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    }
    
    $logger = Logger::getInstance();
    $logger->debug("Medio de pago desconocido");
    
    if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
        return new PaypalPaymentMethod();
    }
    
    return new DefaultPaymentMethod();
}
```

El contexto habitual de esta técnica es la de tratar casos particulares o que sean obvios en los primeros pasos del algoritmo, volviendo al flujo principal cuanto antes, de modo que solo recibe aquellos casos a los que se aplica realmente. Una variante de esta idea consiste en la introducción de las cláusulas de guarda, que veremos a continuación.

### Cláusulas de guarda

En muchas ocasiones, cuando los datos tienen que ser validados antes de operar con ellos, podemos encapsular esas condiciones en forma de cláusulas de guarda. Estas cláusulas de guarda, también se conocen como aserciones, o precondiciones. Si los parámetros recibidos no las cumplen, el método o función falla, generalmente, lanzando excepciones.

```php
public function applyDiscount($parameter)
{
    if ($parameter > 100 || $parameter < 0) {
        throw new OutOfRangeException(sprintf('Parameter should be between 0 and 100 (inc), %s provided.', $parameter));
    }
    
    // further processing
}
```

Extraemos toda la estructura a un método privado:

```php
public function applyDiscount($parameter)
{
    $this->checkParameterIsInRange($parameter);
        
    // further processing
}

private function checkParameterIsInRange($parameter) 
{
    if ($parameter > 100 || $parameter < 0) {
        throw new OutOfRangeException(sprintf('Parameter should be between 0 and 100 (inc), %s provided.', $parameter));
    }
}
```

La lógica bajo este tipo de cláusulas es que si no salta ninguna excepción, quiere decir que `$parameter` ha superado todas las validaciones y lo puedes usar con confianza. La ventaja es que las reglas de validación definidas con estas técnicas resultan muy expresivas, ocultando los detalles técnicos en los métodos extraídos.

Una alternativa es usar una librería de aserciones, lo que nos permite hacer lo mismo de una forma aún más limpia y reutilizable. Si la aserción no se cumple, se tirará una excepción:

```php
public function applyDiscount($parameter)
{
    Assert::betweenExclusive($parameter, 0, 100);
        
    // further processing
}

```

Una limitación de las aserciones que debemos tener en cuenta es que no sirven para control de flujo. Esto es, las aserciones fallan con una excepción, interrumpiendo la ejecución del programa, que así puede comunicar al módulo llamante una circunstancia que impide continuar. Por eso, las aserciones son ideales para validar precondiciones.

En el caso de necesitar una alternativa si el parámetro no cumple los requisitos, utilizaremos condicionales. Por ejemplo, si el parámetro excede los límites queremos que se ajuste al límite que ha superado, en vez de fallar: 

```php
$parameter = $this->checkTheParameterIsInRange($parameter);

// further processing

private function checkTheParameterIsInRange(int $parameter)
{
    if ($parameter > 100) {
        return 100;
    }
    
    if ($parameter < 0) {
        return 0;
    }
    
    return $parameter;
}
```

Finalmente, ten en cuenta que el tipado ya es una guarda en sí misma, por lo que no necesitas verificar el tipo de un parámetro si ya lo has tipado correctamente.

## Preferir condiciones afirmativas

Diversos estudios han mostrado que las frases afirmativas son más fáciles de entender que las negativas, por lo que siempre que sea posible deberíamos intentar convertir la condición en afirmativa ya sea invirtiéndola, ya sea encapsulándola de modo que se exprese de manera afirmativa. Con frecuencia, además, la particular sintaxis de la negación puede hacerlas poco visibles:

```php
if (!$selectedPaymentMethod) {
    return $selectedPaymentMethod;
} 
```

En uno de los ejemplos anteriores habíamos llegado a la siguiente construcción invirtiendo la condicional lo que resultó en una doble negación que puede hacerse difícil de leer:

```php
if (null !== $selectedPaymentMethod) {
    return $selectedPaymentMethod;
} 
```

Nosotros lo que queremos es devolver el método de pago en caso de tener uno seleccionado:

```php
if ($selectedPaymentMethod) {
    return $selectedPaymentMethod;
} 
```

Una forma alternativa, si la condición es compleja o simplemente difícil de entender tal cual es encapsularla en un método:

```php
if ($this->userHasSelectedAPaymentMethod($selectedPaymentMethod)) {
    return $selectedPaymentMethod;
} 

function userHasSelectedAPaymentMethod($selectedPaymentMethod)
{
    return null !== $selectedPaymentMethod;
}
```

## Encapsula expresiones complejas en métodos o funciones

La idea es encapsular expresiones condicionales complejas en funciones o métodos, de modo que su nombre describa el significado de la expresión condicional, manteniendo ocultos los detalles _escabrosos_ de la misma. Esto puede hacerse de forma global o por partes.

Justo en el apartado anterior hemos visto un ejemplo de esto mismo, haciendo explícito el significado de una expresión condicional difícil de leer.

Veamos otro caso en el mismo ejemplo: la extraña condicional que selecciona el método de pago en función del país de destino y el número de pedido. Posiblemente, algún apaño para resolver un problema concreto en los primeros pasos de la empresa.

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    }
    
    $logger = Logger::getInstance();
    $logger->debug("Medio de pago desconocido");
    
    if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
        return new PaypalPaymentMethod();
    }
    
    return new DefaultPaymentMethod();
}
```

Se podría encapsular la condición en un método que sea un poco más explicativo:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    }
    
    $logger = Logger::getInstance();
    $logger->debug("Medio de pago desconocido");
    
    if ($this->isLegacyOrderWithDestinationFrance($order)) {
        return new PaypalPaymentMethod();
    }
    
    return new DefaultPaymentMethod();
}
```

Y el método encapsulado sería algo así:

```php
private function isLegacyOrderWithDestinationFrance($order)
{
    return $order->getDestinationCountry() == Country::FRANCE && $order->id() < 745;
}
```

## Encapsula ramas en métodos o funciones

Consiste en encapsular todo el bloque de código de cada rama de ejecución en su propio método, de modo que el nombre nos indique qué hace. Esto nos deja las ramas de la estructura condicional al mismo nivel y expresando lo que hacen de manera explícita y global. En los métodos extraídos podemos seguir aplicando refactors progresivos hasta que ya no sea necesario.

Este fragmento de código, que está bastante limpio, podría clarificarse un poco, encapsulando tanto las condiciones como la rama:

```php
if ($productStatus == OrderStatuses::PROVIDER_PENDING ||
    $productStatus == OrderStatuses::PENDING ||
    $productStatus == OrderStatuses::WAITING_FOR_PAYMENT
) {
    if ($paymentMethod == PaymentTypes::BANK_TRANSFER) {
        return 'pendiente de transferencia';
    }
    if ($paymentMethod == new PaypalPaymentMethod() || $paymentMethod == PaymentTypes::CREDIT_CARD) {
        return 'pago a crédito';
    }
    if ($this->paymentMethods->hasSelectedDebitCard()) {
        return 'pago a débito';
    }
    if (!$this->paymentMethods->requiresAuthorization()) {
        return 'pago no requiere autorización';
    }
}
```

Veamos como:

```php
if ($this->productIsInPendingStatus($productStatus)) {
    return $this->reportForProductInPendingStatus($paymentMethod);
}

private function productIsInPendingStatus($productStatus)
{
    return ($productStatus == OrderStatuses::PROVIDER_PENDING ||
    $productStatus == OrderStatuses::PENDING ||
    $productStatus == OrderStatuses::WAITING_FOR_PAYMENT);
}

private function reportForProductInPendingStatus(paymentMethod)
{
    if ($paymentMethod == PaymentTypes::BANK_TRANSFER) {
        return 'pendiente de transferencia';
    }
    if ($paymentMethod == new PaypalPaymentMethod() || $paymentMethod == PaymentTypes::CREDIT_CARD) {
        return 'pago a crédito';
    }
    if ($this->paymentMethods->hasSelectedDebitCard()) {
        return 'pago a débito';
    }
    if (!$this->paymentMethods->requiresAuthorization()) {
        return 'pago no requiere autorización';
    }
}
```

De ese modo, la complejidad queda oculta en los métodos y el cuerpo principal se entiende fácilmente. Ya es cuestión nuestra si necesitamos seguir el refactor dentro de los métodos privados que acabamos de crear.

### _Equalize branches_

Si hacemos esto en todas las ramas de una condicional o de un switch las dejaremos todas al mismo nivel, lo que facilita su lectura.

## Reemplaza `if…elseif` sucesivos con `switch`

En muchos casos, sucesiones de `if` o `if…else` quedarán mejor expresados mediante una estructura `switch`. Por ejemplo, siguiendo con el ejemplo anterior, este método que hemos extraído:

```php
private function reportForProductInPendingStatus(paymentMethod)
{
    if ($paymentMethod == PaymentTypes::BANK_TRANSFER) {
        return 'pendiente de transferencia';
    }
    if ($paymentMethod == new PaypalPaymentMethod() || $paymentMethod == PaymentTypes::CREDIT_CARD) {
        return 'pago a crédito';
    }
    if ($this->paymentMethods->hasSelectedDebitCard()) {
        return 'pago a débito';
    }
    if (!$this->paymentMethods->requiresAuthorization()) {
        return 'pago no requiere autorización';
    }
}
```

Podría convertirse en algo así:

```php
private function reportForProductInPendingStatus(paymentMethod)
{
    switch $paymentMethod {
        case PaymentTypes::BANK_TRANSFER:
            return 'pendiente de transferencia';
        case new PaypalPaymentMethod():
        case PaymentTypes::CREDIT_CARD:
            return 'pago a crédito';
    }
    
    if ($this->paymentMethods->hasSelectedDebitCard()) {
        return 'pago a débito';
    }
    if (!$this->paymentMethods->requiresAuthorization()) {
        return 'pago no requiere autorización';
    }
}
```

## Sustituir `if` por el operador ternario

A veces, un operador ternario puede ser más legible que una condicional:

```php
function selectElement(Criteria $criteria, Desirability $desirability)
{
    $found = false;
    
    $elements = $this->getElements($criteria);
    
    foreach($elements as $element) {
        if (!$found && $this->isDesired($element, $desirability)) {
            $result = $element;
            $found = true;
        }
    }
    if (!$found) {
        $result = null;
    }
    
    return $result;
}
```

Realmente las últimas líneas pueden expresarse en una sola y queda más claro:

```php
function selectElement(Criteria $criteria, Desirability $desirability)
{
    $found = false;
    
    $elements = $this->getElements($criteria);
    
    foreach($elements as $element) {
        if (!$found && $this->isDesired($element, $desirability)) {
            $result = $element;
            $found = true;
        }
    }
    
    return $found ? $result : null;
}
```

El operador ternario tiene sus problemas, pero, en general, es una buena solución cuando queremos expresar un cálculo que se resuelve de dos maneras según una condición. Eso sí: nunca anides operadores ternarios porque su lectura entonces se complica enormemente.

De todos modos, este ejemplo concreto de código puede mejorar mucho. Si nos fijamos, el primer elemento encontrado ya nos basta como respuesta, por lo que se podría devolver inmediatamente. No necesitamos ninguna variable temporal, ni una doble condición. Este tipo de refactor lo veremos en el capítulo sobre _return early_.

```php
function selectElement(Criteria $criteria, Desirability $desirability)
{ 
    foreach($this->getElements($criteria) as $element) {
        if ($this->isDesired($element, $desirability)) {
            return $element;
        }
    }
    
    return null;
}
```

## Solo un `if` por método

Christian Clausen en [Five Lines of Code](https://www.manning.com/books/five-lines-of-code) propone un refactor para condicionales que puede ser muy interesante. Con frecuencia, una estructura condicional indica que una función está haciendo varias cosas distintas. Por tanto, lo que propone es que cada método haga solo una cosa, y que, por tanto, solo tenga un `if` que sería la primera línea. Si hay más de uno, separamos en métodos distintos.

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    }
    
    $logger = Logger::getInstance();
    $logger->debug("Medio de pago desconocido");
    
    if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
        return new PaypalPaymentMethod();
    }
    
    return new DefaultPaymentMethod();
}
```

Podría quedar más o menos así:

```php
function obtainPaymentMethod(Order $order): PaymentMethod {
    $selectedPaymentMethod = $order->getSelectedPaymentMethod();
    if ($selectedPaymentMethod !== null) {
        return $selectedPaymentMethod;
    }
    
    $logger = Logger::getInstance();
    $logger->debug("Medio de pago desconocido");
    
    return obtainDefaultPaymentMethod($order);
}

function obtainDefaultPaymentMethod(Order $order): PaymentMethod {
     if ($order->getDestinationCountry() == Country::FRANCE && $order->id() < 745) {
        return new PaypalPaymentMethod();
     }
    
    return new DefaultPaymentMethod();
}
```

## Resumen del capítulo

Las expresiones y estructuras condicionales pueden hacer que seguir el flujo de un código sea especialmente difícil, particularmente cuando están anidadas o son muy complejas. Mediante técnicas de extracción podemos simplificarlas, aplanarlas y hacerlas más expresivas.

Estas normas generales nos pueden ser útiles:

* Poner la condición más corta primero.
* Encapsular las condiciones en métodos o funciones para expresar la intención.
* Encapsular las ramas en métodos o funciones que expresen la intención y, de paso, simplificar la estructura.
* Evitar else cuando sea posible.
* Expresar las condiciones de forma afirmativa.

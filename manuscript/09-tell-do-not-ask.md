# Aplica Tell, Don't Ask

> En el que tratamos sobre la redistribución de responsabilidades. Porque en un sistema orientado a objetos la pregunta no es: ¿cómo hay que hacer esto?, sino: ¿quién debería estar haciendo esto? Y, por tanto, lo que buscamos es la forma de mover las responsabilidades y comportamientos a los objetos a los que pertenecen naturalmente. 

Hasta ahora, hemos trabajado refactors muy orientados a mejorar la expresividad del código y a la organización de unidades de código. En este capítulo vamos a trabajar en cómo mejorar las relaciones entre objetos.

Los principios de diseño nos proporcionan criterios útiles tanto para guiarnos en el desarrollo como para evaluar código existente en el que tenemos que intervenir. Vamos a centrarnos en dos principios que son bastante fáciles de aplicar y que mejorarán la inteligibilidad y la posibilidad de testear nuestro código. Se trata de *Tell, Don't Ask* y la *Ley de Demeter*.

Primero haremos un repaso y luego los veremos en acción.

## Tell, don't ask

La traducción de este enunciado a español sería algo así como "Ordena, no preguntes". La idea de fondo de este principio es que cuando queremos modificar un objeto basándose su propio estado, no es buena idea preguntarle por su estado (*ask*), hacer el cálculo y cambiar su estado si fuera preciso. En su lugar, lo propio sería encapsular ese proceso en un método del propio objeto y decirle (*tell*) que lo realice él mismo.

Dicho en otras palabras: cada objeto debe ser responsable de su estado.

Veamos un ejemplo bastante absurdo, pero que lo deja claro.

Supongamos que tenemos una clase `Square` que representa un cuadrado y queremos poder calcular su área.

```php
$square = new Square(20);

$side = $square->side();

$area = $side**2;
```

Si aplicamos el principio *Tell, Don't Ask*, el cálculo del área estaría en la clase `Square`:

```php
$square = new Square(20);

$area = $square->area();
```

Mejor, ¿no? Veamos por qué.

En el dominio de las figuras geométricas planas, el área o superficie es una propiedad que tienen todas ellas y que, a su vez, depende de otras que son su base y su altura, que en el caso del cuadrado coinciden. La función para determinar el área ocupada por una figura plana depende de cada figura específica.

Posiblemente, estés de acuerdo en que al modelar este comportamiento lo pondríamos en la clase de cada figura desde el primer momento, lo que seguramente nos llevaría a una interfaz.

```php
interface TwoDimensionalShapeInterface
{
    public function area(): float;
}
```

La primera razón es que toda la información necesaria para calcular el área está en la clase, por lo que tiene todo el sentido del mundo que el conocimiento preciso para calcularla también esté allí. Se aplica el **principio de Cohesión** y el **principio de Encapsulamiento**, manteniendo juntos los datos y las funciones que procesan esos datos.

Una segunda razón es más pragmática: si el conocimiento para calcular el área está en otro lugar, como un servicio, cada vez que necesitemos incorporar una nueva clase al sistema, tendremos que modificar el servicio para añadirle ese conocimiento, rompiendo el **principio Abierto/Cerrado**. 

En tercer lugar, el testing se simplifica. Es fácil hacer tests de las clases que representan las figuras. Por otro lado, el testing de las otras clases que las utilizan también se simplifica. Normalmente, esas clases usarán el cálculo del área como una utilidad para llevar a cabo sus propias responsabilidades que es lo que queremos saber si hacen correctamente.

Siguiendo el principio ***Tell, Don't Ask*** movemos responsabilidades a los objetos a los que pertenecen. 

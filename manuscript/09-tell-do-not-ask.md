# Aplica Tell, Don't Ask

> En el que buscamos empujar el comportamiento dentro de nuestros objetos para que sepan hacer cosas por sí mismos, en lugar de preguntarles por lo que saben. 

Hasta ahora, hemos trabajado refactors muy orientados a mejorar la expresividad del código y a la organización de unidades de código. Pero en estos capítulos nos enfocaremos en los principios de diseño orientado a objetos.

Los principios de diseño nos proporcionan criterios útiles tanto para guiarnos en el desarrollo como para evaluar código existente en el que tenemos que intervenir.

### Notas de la segunda edición

En esta revisión, hemos dividido este capítulo en dos a fin de tratar de forma más detallada los principios presentados: _Tell, Don't Ask_ y la _Ley de Demeter_. Esto nos permitirá explorarlos con más detalle y mejores ejemplos.

## Tell, don't ask

La traducción de este enunciado a español sería algo así como "Pide, no preguntes". La idea de fondo de este principio es que cuando queremos modificar un objeto basándose su propio estado, no es buena idea preguntarle por su estado (*ask*), hacer el cálculo y cambiar su estado si fuera preciso. En su lugar, lo propio sería encapsular ese proceso en un método del propio objeto y decirle (*tell*) que lo realice él mismo.

Dicho en otras palabras: cada objeto debe ser responsable de su estado representado por sus propiedades internas, manteniéndolo oculto a los demás objetos, que solo conocerán su interfaz pública.

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

En el dominio de las figuras geométricas planas, el área o superficie es una propiedad que tienen todas ellas y que, a su vez, depende de su base y su altura, que en el caso del cuadrado coinciden. La función para determinar el área ocupada por una figura plana es diferente para cada tipo de figura.

Posiblemente, estés de acuerdo en que al modelar este comportamiento lo pondríamos en la clase de cada figura desde el primer momento, lo que seguramente nos llevaría a una interfaz.

```php
interface TwoDimensionalShape
{
    public function area(): float;
}
```

La primera razón es que toda la información necesaria para calcular el área está en la clase, por lo que tiene todo el sentido del mundo que el conocimiento preciso para calcularla también esté allí. Se aplica el **principio de Cohesión** y el **principio de Encapsulamiento**, manteniendo juntos los datos y las funciones que procesan esos datos.

Una segunda razón es más pragmática: si el conocimiento para calcular el área está en otro lugar, como un servicio, cada vez que necesitemos incorporar una nueva clase al sistema, tendremos que modificar el servicio para añadirle ese conocimiento, rompiendo el **principio Abierto/Cerrado**. 

En tercer lugar, el testing se simplifica. Es fácil hacer tests de las clases que representan las figuras. Por otro lado, el testing de las otras clases que las utilizan también se simplifica. Normalmente, esas clases usarán el cálculo del área como una utilidad para llevar a cabo sus propias responsabilidades que es lo que queremos saber si hacen correctamente.

Siguiendo el principio ***Tell, Don't Ask*** movemos responsabilidades a los objetos a los que pertenecen. 

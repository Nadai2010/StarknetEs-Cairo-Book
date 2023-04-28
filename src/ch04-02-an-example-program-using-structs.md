# Un programa de ejemplo usando estructuras

Para entender cuándo podríamos usar estructuras, escribamos un programa que calcule el área de un rectángulo. Comenzaremos usando variables individuales y luego reescribiremos el programa hasta que estemos usando estructuras en su lugar.

Hagamos un nuevo proyecto con Scarb llamado *rectangles* que tomará el ancho y la altura de un rectángulo en píxeles y calculará el área del rectángulo. El Listado 4-6 muestra un pequeño programa con una forma de hacer exactamente eso en el *src/lib.cairo* de nuestro proyecto.

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;
fn main() {
    let width1 = 30_u64;
    let height1 = 10_u64;
    let area = area(width1, height1);
    area.print();
}

fn area(width: u64, height: u64) -> u64 {
    width * height
}
```

<span class="caption">Listado 4-6: Cálculo del área de un rectángulo especificado por variables separadas de ancho y alto</span>

Para compilar el programa usamos `cairo-run src/lib.cairo`:

```bash
$ cairo-run src/lib.cairo
[DEBUG] ,                               (raw: 300)

Run completed successfully, returning []
```

Este código logra calcular el área del rectángulo llamando a la función `area` con cada dimensión, pero podemos hacer más para que este código sea claro y legible.

El problema con este código es evidente en la declaración de la función `area`:

```rust
fn area(width: u64, height: u64) -> u64 {
```

Se supone que la función `area` calcula el área de un rectángulo, pero la función que escribimos tiene dos parámetros, y no está claro en ninguna parte de nuestro programa que los parámetros estén relacionados. Sería más legible y manejable agrupar el ancho y el alto juntos. Ya discutimos una forma en que podríamos hacer eso en el [Capítulo 3](ch02-02-data-types.html#the-tuple-type): usando tuplas.


## Reescribiendo con tuplas

El listado 4-7 muestra otra versión de nuestro programa usando tuplas.

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;
fn main() {
    let rectangle = (30_u64, 10_u64);
    let area = area(rectangle);
    area.print(); // print out the area
}

fn area(dimension: (u64, u64)) -> u64 {
    let (x,y) = dimension;
    x * y
}
```

<span class="caption">Listing 4-7: Especificando el ancho y alto de un rectangulo con una tupla</span>

En cierto modo, este programa es mejor. Las tuplas nos permiten agregar un poco de estructura y ahora estamos pasando solo un argumento. Pero en otro sentido, esta versión es menos clara: las tuplas no nombran sus elementos, por lo que tenemos que indexar las partes de la tupla, lo que hace que nuestro cálculo sea menos obvio.

Mezclar el ancho y la altura no importaría para el cálculo del área, pero si queremos calcular la diferencia, ¡sería importante! Tendríamos que tener en cuenta que `width` es el índice de tupla `0` y `height` es el índice de tupla `1`. Esto sería aún más difícil de entender y tener en cuenta para otra persona si usara nuestro código. Debido a que no hemos transmitido el significado de nuestros datos en nuestro código, ahora es más fácil introducir errores.



## Reescribiendo con `struct`: agrega más significado

Usamos estructuras para agregar significado al etiquetar los datos. Podemos transformar la tupla que estamos usando en una estructura con un nombre para el todo y nombres para las partes.


<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;

struct Rectangle {
    width: u64,
    heigh: u64,
}

fn main() {
    let rectangle = Rectangle {
        width: 30_u64,
        heigh: 10_u64,
    };
    let area = area(rectangle);
    area.print(); // print out the area
}

fn area(rectangle: Rectangle) -> u64 {
    rectangle.width * rectangle.heigh
}
```

<span class="caption">Listado 4-8: Definición de una estructura llamada `Rectangle`</span>

Aquí hemos definido una estructura y la hemos llamado `Rectangle`. Dentro de las llaves, definimos los campos como `width` y `height`, los cuales tienen el tipo `u64`. Luego, en `main`, creamos una instancia particular de `Rectangle` que tiene un ancho de `30` y una altura de `10`. Nuestra función `area` ahora está definida con un parámetro, al que hemos llamado `rectangle` que es de tipo de la estructura `Rectangle`. Luego podemos acceder a los campos de la instancia con notación de punto, y dar nombres descriptivos a los valores en lugar de usar los valores de índice de tupla de `0` y `1`.


## Agregando funcionalidades útiles con `trait`

Sería útil poder imprimir una instancia de `Rectangle` mientras estamos depurando nuestro programa y ver los valores de todos sus campos. El Listado 4-9 intenta usar `print` como lo hemos usado en capítulos anteriores. Esto no funcionará.

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;

struct Rectangle {
    width: u64,
    heigh: u64,
}

fn main() {
    let rectangle = Rectangle {
        width: 30_u64,
        heigh: 10_u64,
    };
    rectangle.print();
}
```

<span class="caption">Listado 4-9: Intentando imprimir una instancia de `Rectangle`</span>

Cuando compilamos este código, obtenemos un error con el siguiente mensaje:

```bash
$ cairo-compile src/lib.cairo
error: Method `print` not found on type "../src::Rectangle". Did you import the correct trait and impl?
 --> lib.cairo:16:15
    rectangle.print();
              ^***^

Error: Compilation failed.
```

El trait `print` está implementado para muchos tipos de datos, pero no para la estructura `Rectangle`. Podemos arreglar esto implementando el trait `PrintTrait` en la estructura `Rectangle` como se muestra en el Listado 4-10.

Para aprender más sobre traits,[Traits en Cairo](ch07-02-traits-in-cairo.md).

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;

struct Rectangle {
    width: u64,
    heigh: u64,
}

fn main() {
    let rectangle = Rectangle {
        width: 30_u64,
        heigh: 10_u64,
    };
    rectangle.print();
}

impl RectanglePrintImpl of PrintTrait<Rectangle> {
    fn print(self: Rectangle) {
        self.width.print();
        self.heigh.print();
    }
}
```

<span class="caption">Listado 4-10:  Implementación del trait `PrintTrait` en `Rectangle`</span>

¡Bien! No es el resultado más bonito, pero muestra los valores de todos los campos para esta instancia, lo que definitivamente ayudaría durante la depuración.

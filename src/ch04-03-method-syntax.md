## Sintaxis de métodos

Los métodos son similares a las funciones: los declaramos con la palabra clave `fn` y un nombre, pueden tener parámetros, retornar un valor, y contener código que se ejecuta cuando el método es llamado desde otro lugar. A diferencia de las funciones, los métodos se definen dentro del contexto de un tipo y su primer parámetro siempre es `self`, que representa la instancia del tipo al que se llama el método. Para aquellos familiarizados con Rust, el enfoque de Cairo puede resultar confuso, ya que los métodos no se pueden definir directamente en los tipos. En su lugar, debe definir un `trait` y una implementación asociados con el tipo para el que está destinado el método.

### Definición de métodos

Cambiemos la función `area` que tiene una instancia de `Rectangle` como parámetro y en su lugar crea un método `area` definido en el *trait* `RectangleTrait`, como se muestra en el Listado 4-13. 

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;
#[derive(Copy, Drop)]
struct Rectangle {
    width: u64,
    height: u64,
}

trait RectangleTrait {
    fn area(self: @Rectangle) -> u64;
}

impl RectangleImpl of RectangleTrait {
    fn area(self: @Rectangle) -> u64 {
        (*self.width) * (*self.height)
    }
}

fn main() {
    let rect1 = Rectangle { width: 30_u64, height: 50_u64,  };

    rect1.area().print();
}
```

<span class="caption">Listado 4-13: Definiendo el método `area` para usar en la estructura `Rectangle` </span>

Para definir la función dentro del contexto de `Rectangle`, comenzamos definiendo un `trait` con la declaración del método que queremos implementar. Los *Traits* no están vinculados a un tipo específico; solo el parámetro `self` del método define qué tipo se puede usar con dicho *trait*. Luego, definimos un bloque con la palabra clave `impl` para `RectangleTrait`, que define el comportamiento de los métodos implementados. Todo dentro de este bloque `impl` será asociado con el tipo del parámetro `self` del método llamado. Si bien es técnicamente posible definir métodos para múltiples tipos dentro del mismo bloque `impl`, no es una práctica recomendada, ya que puede producir una confusión. Recomendamos que el tipo del parámetro `self` permanece consistente dentro del mismo bloque `impl`. Luego movemos la función `area` dentro de los corchetes `impl` y cambiamos el primer (y en este caso, único) parámetro para ser `self` en la declaración y en todas partes dentro del cuerpo. En `main`, donde llamamos a la función `area` y pasamos `rect1` como argumento, en su lugar, podemos usar la *sintaxis del método* para llamar al método `area` en nuestra instancia del `Rectangle`. La sintaxis del método va después de una instancia: agregamos un punto seguido del nombre del método, los paréntesis y los argumentos.

Los métodos deben tener un parámetro llamado `self` del tipo al que se aplicarán para su primer parámetro. Tenga en cuenta que usamos el operador `@` (*snapshot*) delante del tipo `Rectangle` en la declaración de la función. Al hacerlo, indicamos que este método toma un *snapshot* inmutable de la instancia de `Rectangle`, que es creado automáticamente por el compilador al pasar la instancia al método. Los métodos pueden tomar posesión de `self`, usar `self` con *snapshot* como lo hemos hecho aquí, o usar una referencia mutable a `self`
    utilizando la sintaxis `ref self: T`.

Elegimos `self: @Rectangle` por la misma razón que usamos `@Rectangle` en la función versión: no queremos tomar posesión, y solo queremos leer los datos en la estructura, no escribir en ella. Si quisiéramos cambiar la instancia que hemos llamado al método como parte de lo que hace el método, usaríamos `ref self: Rectangle` como el primer parámetro. Tener un método que tome posesión de la instancia por usar solo `self` como primer parámetro es raro; esta técnica suele ser usada cuando el método transforma `self` en otra cosa y desea evitar que la persona que llama use la instancia original después de la transformación.

Observe el uso del operador *desnap* `*` dentro del método *area* cuando accede a los miembros de la estructura. Esto es necesario porque la estructura se pasa como una *snapshot* y todos sus valores de campo son del tipo `@T`, requiriendo que sean *desnapped* para poder manipularlos.

La principal razón para usar métodos en lugar de funciones es la organización y la claridad del código. Hemos puesto todas las cosas que podemos hacer con una instancia de un tipo en una combinación de bloques de tipo `trait` &amp; `impl`, en lugar de hacer que los futuros usuarios de nuestro código busquen capacidades de `Rectangle` en varios lugares en la biblioteca que ofrecemos. Sin embargo, podemos definir múltiples combinaciones de `trait` &amp; `impl` para el mismo tipo en diferentes lugares, lo que puede ser útil para organizar nuestro código. Por ejemplo, podría implementar el *trait* `Add` para su tipo en un bloque `impl`, y el *trait* `Sub` en otro bloque.

Tenga en cuenta que podemos optar por dar a un método el mismo nombre que uno de los campos de la estructura. Por ejemplo, podemos definir un método en `Rectangle` que también se llama
    `width`:

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;
#[derive(Copy, Drop)]
struct Rectangle {
    width: u64,
    height: u64,
}

trait RectangleTrait {
    fn width(self: @Rectangle) -> bool;
}

impl RectangleImpl of RectangleTrait {
    fn width(self: @Rectangle) -> bool {
        (*self.width) > 0_u64
    }
}

fn main() {
    let rect1 = Rectangle { width: 30_u64, height: 50_u64,  };
    rect1.width().print();
}
```

Aquí, elegimos hacer que el método `width` devuelva `true` si el valor en el campo `width` de la instancia es mayor que `0` y `false` si el valor es `0`: podemos usar un campo dentro de un método del mismo nombre para cualquier propósito. En `main`, cuando colocamos `rect1.width` entre paréntesis, Cairo sabe que nos referimos al método `width`. Cuando no usamos paréntesis, Cairo sabe que nos referimos al campo `widht`.

### Métodos con más parámetros

Practiquemos el uso de métodos implementando un segundo método en la estructura `Rectangle`. Esta vez queremos que una instancia de `Rectangle` tome otra instancia de `Rectangle` y devolver `true` si el segundo `Rectangle` puede caber completamente dentro de `self` (el primer `Rectangle`); de lo contrario, debería devolver `false`. Es decir, una vez que hemos definido el método `can_hold`, queremos poder escribir el programa que se muestra en el Listado 4-14.

<span class="filename">Filename: src/lib.cairo</span>

```swift
use debug::PrintTrait;
#[derive(Copy, Drop)]
struct Rectangle {
    width: u64,
    height: u64,
}


fn main() {
    let rect1 = Rectangle {
        width: 30_u64,
        height: 50_u64,
    };
    let rect2 = Rectangle {
        width: 10_u64,
        height: 40_u64,
    };
    let rect3 = Rectangle {
        width: 60_u64,
        height: 45_u64,
    };

    'Can rect1 hold rect2?'.print();
    rect1.can_hold(@rect2).print();

    'Can rect1 hold rect?'.print();
    rect1.can_hold(@rect3).print();
}
```

<span class="caption">Listing 4-14:  Usando el método todavia no escrito `can_hold`</span>

La salida esperada sería similar a la siguiente porque ambas dimensiones de `rect2` son más pequeñas que las dimensiones de `rect1`, pero `rect3` es más ancha que` rect1`:

```text
❯ cairo-run src/lib.cairo
[DEBUG]	Can rec1 hold rect2?           	(raw: 384675147322001379018464490539350216396261044799)

[DEBUG]	true                           	(raw: 1953658213)

[DEBUG]	Can rect1 hold rect?           	(raw: 384675147322001384331925548502381811111693612095)

[DEBUG]	false                          	(raw: 439721161573)

```

We know we want to define a method, so it will be within the `trait RectangleTrait`
and `impl RectangleImpl of RectangleTrait` blocks.
The method name will be `can_hold`, and it will take a snapshot
of another `Rectangle` as a parameter. We can tell what the type of the
parameter will be by looking at the code that calls the method:
`rect1.can_hold(@rect2)` passes in `@rect2`, which is a snapshot to
`rect2`, an instance of `Rectangle`. This makes sense because we only need to
read `rect2` (rather than write, which would mean we’d need a mutable borrow),
and we want `main` to retain ownership of `rect2` so we can use it again after
calling the `can_hold` method. The return value of `can_hold` will be a
Boolean, and the implementation will check whether the width and height of
`self` are greater than the width and height of the other `Rectangle`,
respectively. Let’s add the new `can_hold` method to the `trait` and `impl` blocks from
Listing 4-13, shown in Listing 4-15.

Sabemos que queremos definir un método, por lo que estará dentro del bloque `trait RectangleTrait` e `impl RectangleImpl of RectangleTrait`. El nombre del método será `can_hold`, y tomará una *snapshot* de otro `Rectangle` como parámetro. Podemos decir cuál será el tipo de parámetro si miramos el código que llama al método: `rect1.can_hold(@rect2)` pasa `@rect2`, que es un *snapshot* para `rect2`, una instancia de `Rectangle`. Esto tiene sentido porque solo necesitamos leer `rect2` (en lugar de escribir, lo que significaría que necesitaríamos un préstamo mutable), y queremos que `main` conserve la propiedad de `rect2` para que podamos usarlo nuevamente después llamando al método `can_hold`. El valor de retorno de `can_hold` será un Boolean, y la implementación verificará si el ancho y la altura de `self` son mayores que el ancho y alto del otro `Rectangle`, respectivamente. Agreguemos el nuevo método `can_hold` a los bloques `trait` y `impl` del Listado 4-13, mostrado en el Listado 4-15.

<span class="filename">Filename: src/lib.cairo</span>

```rust
trait RectangleTrait {
    fn area(self: @Rectangle) -> u64;
    fn can_hold(self: @Rectangle, other: @Rectangle) -> bool;
}

impl RectangleImpl of RectangleTrait {
    fn area(self: @Rectangle) -> u64 {
        *self.width * *self.height
    }

    fn can_hold(self: @Rectangle, other: @Rectangle) -> bool {
        *self.width > *other.width & *self.height > *other.height
    }
}
```

<span class="caption">Listing 4-15: Implementación del método `can_hold` en `Rectangle` que recibe una instancia de `Rectangle` como parámetro</span>

When we run this code with the `main` function in Listing 5-14, we’ll get our
desired output. Methods can take multiple parameters that we add to the
signature after the `self` parameter, and those parameters work just like
parameters in functions.

Cuando ejecutamos este código con la función `main` en el Listado 4-14, obtendremos nuestra salida deseada. Los métodos pueden tomar múltiples parámetros que agregamos a su definición después del parámetro `self`, y esos parámetros funcionan como parámetros en las funciones.

### Acceso a las funciones de implementación

Todas las funciones definidas dentro de un bloque `trait` e `impl` se pueden llamar directamente utilizando el operador `::` en el nombre de la implementación. Las funciones en los *trait* que no son métodos a menudo se usan para constructores que devolverá una nueva instancia de la estructura. Estos a menudo se denominan `new`, pero `new` no es un nombre especial y no está integrado en el lenguaje de Cairo. Por ejemplo, nosotros podriamos optar por proporcionar una función asociada llamada `square` que tendría un parámetro de dimensión y usarlo como ancho y alto, haciéndolo así más fácil para crear un cuadrado con `Rectangle` en lugar de tener que especificar el mismo valor dos veces:

<span class="filename">Filename: src/lib.cairo</span>

```rust
trait RectangleTrait {
    fn square(size:u64) -> Rectangle;
}

impl RectangleImpl of RectangleTrait {
    fn square(size: u64) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

Para llamar a esta función, usamos la sintaxis `::` con el nombre de implementación; por ejemplo, `let square = RectangleImpl::square(10_u64);`. Esta función está espaciada por la implementación: la sintaxis `::` se usa tanto para funciones del *trait* y espacios de nombres creados por módulos. Lo discutiremos en el [Capítulo 7][modules]<!-- ignore -->.

> Note: It is also possible to call this function using the trait name, with `RectangleTrait::square(10_u64)`.

> Nota: También es posible llamar a esta función usando el nombre del *trait*, con `RectangleTrait::square(10_u64)`.


### Multiples bloques con `impl`

Cada estructura tiene permitido tener múltiples bloques con `trait` e `impl`. Por ejemplo, en el Listado 4-15 es equivalente al código mostrado en el Listado 4-16, que tiene cada método en su propio bloque de `trait` e `impl`.

```rust
trait RectangleCalc {
    fn area(self: @Rectangle) -> u64;
}
impl RectangleCalcImpl of RectangleCalc {
    fn area(self: @Rectangle) -> u64 {
        (*self.width) * (*self.height)
    }
}

trait RectangleCmp {
    fn can_hold(self: @Rectangle, other: @Rectangle) -> bool;
}

impl RectangleCmpImpl of RectangleCmp {
    fn can_hold(self: @Rectangle, other: @Rectangle) -> bool {
        *self.width > *other.width & *self.height > *other.height
    }
}
```

<span class="caption">Listado 4-16: Reescribiendo el Listado 4-15 usando múltiples bloques de `impl`</span>


No hay razón para separar estos métodos en múltiples bloques de `trait` e `impl`, pero es una sintaxis válida. 
Veremos un caso en el que usar múltiples bloques es adecuado en el [Capítulo 7](ch07-00-generic-types-and-traits.md), donde discutimos tipos y <em>traits</em> genéricos.

## Resumen 

Las estructuras permiten crear tipos personalizados que son significativos para su dominio. Usando estructuras, puede mantener partes de datos asociadas conectadas entre sí y nombra cada pieza para que tu código quede claro. En los bloques `trait` e `impl`, puedes definir métodos, que son funciones asociadas a un tipo y le permiten especificar el comportamiento que las instancias de su tipo pueden tener.

Pero las estructuras (`struct`) no son la única manera de crear tipos personalizados: pasemos a la función de enumeración (`enum`) de Cairo para agregar otra herramienta.
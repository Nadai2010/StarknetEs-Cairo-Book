## Control de flujo

La capacidad de ejecutar cierto código dependiendo de si una condición es verdadera y de ejecutar código repetidamente mientras una condición es verdadera son bloques de construcción básicos en la mayoría de los lenguajes de programación. Las construcciones más comunes que le permiten controlar el flujo de ejecución del código en Cairo son las expresiones if y los bucles.

### Expresiones `if`

Una expresión if le permite ramificar su código según condiciones. Proporciona una condición y luego establece: "Si se cumple esta condición, ejecute este bloque de código. Si no se cumple la condición, no ejecute este bloque de código".

<span class="filename">Nombre de archivo: main.cairo</span>

```rust
use debug::PrintTrait;

fn main() {
    let number = 3;

    if number == 5 {
        'condition was true'.print();
    } else {
        'condition was false'.print();
    }
}
```

Todos las expresiones `if` comienzan con la palabra clave `if`, seguido de una condición. En este caso, la condición verifica si la variable `number` tiene un valor igual a 5. Colocamos el bloque de código a ejecutar si la condición es `true` inmediatamente después de la condición dentro de llaves.

Opcionalmente, también podemos incluir una expresión `else`, que elegimos hacer aquí, para dar al programa un bloque de código alternativo para ejecutar si la condición se evalúa como `false`. Si no proporciona una expresión `else` y la condición es `false`, el programa simplemente omitirá el bloque `if` y pasará al siguiente fragmento de código.

Intente ejecutar este código; debería ver la siguiente salida:

```console
$ cairo-run main.cairo
[DEBUG]	condition was false
```

Intentaré cambiar el valor de number por uno que haga que la condición sea verdadera para ver qué sucede:

```rust
    let number = 5;
```

```console
$ cairo-run main.cairo
condition was true
```

También vale la pena señalar que la condición en este código debe ser un bool. Si la condición no es un bool, obtendremos un error.

```console
$ cairo-run main.cairo
thread 'main' panicked at 'Failed to specialize: `enum_match<felt252>`. Error: Could not specialize libfunc `enum_match` with generic_args: [Type(ConcreteTypeId { id: 1, debug_name: None })]. Error: Provided generic argument is unsupported.', crates/cairo-lang-sierra-generator/src/utils.rs:256:9
```

### Manejando múltiples condiciones con `else if`

Puede usar múltiples condiciones combinando `if` y `else` en una expresión `else if`. Por ejemplo:

<span class="filename">Nombre del archivo: main.cairo</span>

```rust
use debug::PrintTrait;

fn main() {
    let number = 3;

    if number == 12 {
        'number is 12'.print();
    } else if number == 3 {
        'number is 3'.print();
    } else if number - 2 == 1 {
        'number minus 2 is 1'.print();
    } else {
        'number not found'.print();
    }
}
```

Este programa tiene cuatro posibles caminos que puede seguir. Después de ejecutarlo, debería ver la siguiente salida:

```console
[DEBUG]	number is 3
```

Cuando este programa se ejecuta, verifica cada expresión `if` en orden y ejecuta el primer cuerpo para el cual la condición se evalúa como verdadera. Es importante destacar que aunque `number - 2 == 1` es verdadero, no vemos la salida `number minus 2 is 1'.print()`, ni tampoco vemos el texto `number is not divisible by 4, 3, or 2` del bloque `else`. Esto se debe a que Cairo solo ejecuta el bloque correspondiente a la primera condición verdadera que encuentra, y una vez que la encuentra, no verifica las demás. Usar demasiadas expresiones `else if` puede desordenar el código, por lo que si tienes más de una, es posible que desees refactorizar el código. El capítulo 5 describe una poderosa estructura de control de flujo de Cairo llamada `match` para estos casos.

### Usando `if` en una declaración `let`

Dado que `if` es una expresión, podemos usarla en el lado derecho de una declaración `let` para asignar el resultado a una variable.

<span class="filename">Nombre del archivo: main.cairo</span>

```rust
use debug::PrintTrait;

fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    if number == 5 {
        'condition was true'.print();
    }
}
```

```console
$ cairo-run main.cairo
[DEBUG]	condition was true
```

La variable `number` quedará ligada a un valor basado en el resultado de la expresión `if`. En este caso, será 5.

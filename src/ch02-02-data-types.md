## Tipos de datos

Cada valor en Cairo tiene un cierto _tipo de dato_, lo que le dice a Cairo qué tipo de datos se están especificando para que sepa cómo trabajar con esos datos. Esta sección cubre dos subconjuntos de tipos de datos: escalares y compuestos.

Tenga en cuenta que Cairo es un lenguaje _de tipado estático_, lo que significa que debe conocer los tipos de todas las variables en tiempo de compilación. El compilador suele inferir el tipo deseado en función del valor y su uso. En casos en que pueden ser posibles varios tipos, podemos utilizar un método de conversión donde especificamos el tipo de salida deseado.


```Rust
use traits::TryInto;
use option::OptionTrait;
fn main(){
    let x = 3;
    let y:u32 = x.try_into().unwrap();
}
```

Verá diferentes anotaciones de tipo para otros tipos de datos.

### Tipos escalares

Un tipo _scalar_ representa un único valor. Cairo tiene tres tipos escalares primarios:
fieltros, enteros y booleanos. Puede que reconozca de otros lenguajes de programación. Veamos cómo funcionan en Cairo.

#### Tipo Felt 

En Cairo, si no especificas el tipo de una variable o argumento, su tipo por defecto es un elemento de campo, representado por la palabra clave `felt252`. En el contexto de Cairo, cuando decimos "un elemento de campo" nos referimos a un entero en el rango `0 <= x < P`, donde `P` es un número primo muy grande actualmente igual a `P = 2^{251} + 17 * 2^{192}+1`. Al sumar, restar o multiplicar, si el resultado queda fuera del rango especificado del número primo, se produce un desbordamiento y se suma o resta un múltiplo apropiado de P para que el resultado vuelva a estar dentro del rango (es decir, el resultado se calcula módulo P).

La diferencia más importante entre los números enteros y los elementos de campo es la división: La división de elementos de campo (y, por tanto, la división en Cairo) es distinta de la división normal de las CPU, en la que
la división entera `x / y` se define como `[x/y]` donde se devuelve la parte entera del cociente (por lo que se obtiene `7 / 3 = 2`) y puede o no satisfacer la ecuación `(x / y) * y == x`, dependiendo de la divisibilidad de `x` por `y`.

En Cairo, el resultado de `x/y` está definido para satisfacer siempre la ecuación `(x / y) * y == x`. Si `y` divide a `x` entre enteros, obtendrás el resultado esperado en Cairo (por ejemplo `6 / 2` dará como resultado `3`).
Pero cuando `y` no divide a `x`, puedes obtener un resultado sorprendente: Por ejemplo, como `2 * ((P+1)/2) = P+1 ≡ 1 mod[P]`, el valor de `1 / 2` en Cairo es `(P+1)/2` (y no 0 ó 0,5), ya que satisface la ecuación anterior.
#### Tipos enteros

El tipo felt252 es un tipo fundamental que sirve como base para la creación de todos los tipos en la librería central.
Sin embargo, se recomienda encarecidamente a los programadores que utilicen los tipos enteros en lugar del tipo `felt252` siempre que sea posible, ya que los tipos `integer` vienen con características de seguridad añadidas que proporcionan protección extra contra posibles vulnerabilidades en el código, como comprobaciones de desbordamiento. Utilizando estos tipos de enteros, los programadores pueden asegurarse de que sus programas son más seguros y menos susceptibles a ataques u otras amenazas de seguridad.

Un _integer_ es un número sin componente fraccionario. Esta declaración de tipo indica el número de bits que el programador puede utilizar para almacenar el entero.

La Tabla 3-1 muestra los tipos enteros incorporados en Cairo. Podemos usar cualquiera de estas variantes para declarar
el tipo de un valor entero.

<span class="caption">Table 3-1: Integer Types in Cairo</span>

| Length  | Unsigned |
| ------- | -------- |
| 8-bit   | `u8`     |
| 16-bit  | `u16`    |
| 32-bit  | `u32`    |
| 64-bit  | `u64`    |
| 128-bit | `u128`   |
| 256-bit | `u256`   |
| 32-bit  | `usize`  |

Cada variante tiene un tamaño explícito. Tenga en cuenta que por ahora, el tipo `usize` es sólo un alias para `u32`; sin embargo, podría ser útil cuando en el futuro Cairo pueda ser compilado a MLIR.
Como las variables son sin signo, no pueden contener un número negativo. Este código hará que el programa entre en pánico:

```rust
fn sub_u8s(x: u8, y: u8) -> u8 {
    x - y
}

fn main() {
    sub_u8s(1,3);
}
```

Puede escribir literales enteros en cualquiera de las formas mostradas en la Tabla 3-2. Observe
que los literales numéricos que pueden ser múltiples tipos numéricos permiten un sufijo de tipo
como `57_u8`, para designar el tipo.

<span class="caption">Table 3-2: Integer Literals in Cairo</span>

| Numeric literals | Example   |
| ---------------- | --------- |
| Decimal          | `98222`   |
| Hex              | `0xff`    |
| Octal            | `0o04321` |
| Binary           | `0b01`    |

Entonces, ¿cómo saber qué tipo de entero utilizar? Intenta estimar el valor máximo que puede tener tu int y elige un buen tamaño.
La principal situación en la que usarías `usize` es al indexar algún tipo de colección.

#### Operaciones numéricas

Cairo soporta las operaciones matemáticas básicas que esperarías para todos los tipos de enteros: suma, resta, multiplicación y resto (u256 no soporta división y resto todavía). Entero trunca hacia cero al entero más cercano. El siguiente código muestra cómo utilizar cada operación numérica en una sentencia `let`:

```rust
fn main() {
     // addition
    let sum = 5_u128 + 10_u128;

    // subtraction
    let difference = 95_u128 - 4_u128;

    // multiplication
    let product = 4_u128 * 30_u128;

    // division
    let quotient = 56_u128 / 32_u128; //result is 1
    let quotient = 64_u128 / 32_u128; //result is 2

    // remainder
    let remainder = 43_u128 % 5_u128; // result is 3
}
```

Cada expresión de estas sentencias utiliza un operador matemático y se evalúa a un único valor, que se asigna a una variable.

<!-- TODO: Appendix operator -->
<!-- [Appendix B][appendix_b] ignore contains a list of all operators that Cairo provides. -->

#### El tipo Booleano

Como en la mayoría de los lenguajes de programación, un tipo booleano en Cairo tiene dos posibles valores: `true` y `false`. Los booleanos tienen un byte de tamaño. El tipo booleano en Cairo se especifica usando `bool`. Por ejemplo:

```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

[//]: # "TODO: Control de flujo"

La principal forma de utilizar valores booleanos es a través de condicionales, como una expresión `if` expresión. Cubriremos cómo funcionan las expresiones `if` en Cairo en la sección. [“Control de flujo”][control-de-flujo]<!-- ignore --> 

#### El tipo de Short String

Cairo no tiene un tipo nativo para strings, pero puedes almacenar caracteres formando lo que llamamos un "short string" dentro de `felt252`. Aquí hay algunos ejemplos de declaración de valores entre comillas simples:

```rust
let my_first_char = 'C';
let my_first_string = 'Hello world';
```

### Conversión de Tipos

En Cairo, puedes convertir valores entre tipos escalares comunes y `felt252` usando los métodos `try_into` e `into` proporcionados por los rasgos `TryInto` e `Into`, respectivamente.

El método `try_into` permite una conversión de tipos segura cuando el tipo de destino puede no encajar con el valor de origen. Ten en cuenta que `try_into` devuelve un tipo `Option<T>`, que tendrás que desenvolver para acceder al nuevo valor.

Por otro lado, el método `into` se puede utilizar para la conversión de tipos cuando el éxito está garantizado, como cuando el tipo de destino es más pequeño que el tipo de origen.

Para realizar la conversión, llame a `var.into()` o `var.try_into()` sobre el valor fuente para convertirlo a otro tipo. El tipo de la nueva variable debe definirse explícitamente, como se muestra en el siguiente ejemplo.

```rust
use traits::TryInto;
use traits::Into;
use option::OptionTrait;

fn main(){
    let my_felt = 10;
    let my_u8: u8 = my_felt.try_into().unwrap(); // Since a felt252 might not fit in a u8, we need to unwrap the Option<T> type
    let my_u16: u16 = my_felt.try_into().unwrap();
    let my_u32: u32 = my_felt.try_into().unwrap();
    let my_u64: u64 = my_felt.try_into().unwrap();
    let my_u128: u128 = my_felt.try_into().unwrap();
    let my_u256: u256 = my_felt.into(); // As a felt252 is smaller than a u256, we can use the into() method
    let my_usize: usize = my_felt.try_into().unwrap();
    let my_felt2: felt252 = my_u8.into();
    let my_felt3: felt252 = my_u16.into();
}
```

### Tipos compuestos

_Los tipos compuestos_ pueden agrupar varios valores en un tipo. Cairo tiene dos
tipos compuestos primitivos: Tuplas y Matrices.

#### El tipo tupla

Una _tupla_ es una forma general de agrupar un número de valores con una variedad de tipos en un tipo compuesto. Las tuplas tienen una longitud fija: una vez declaradas, no pueden aumentar ni disminuir de tamaño.

Se crea una tupla escribiendo una lista de valores separados por comas entre paréntesis. Cada posición de la tupla tiene un tipo, y los tipos de los distintos valores de la tupla no tienen por qué ser iguales. Hemos añadido anotaciones opcionales de tipo en este ejemplo:

```rust
fn main() {
    let tup: (u32,u64,bool) = (10,20,true);
}
```

La variable `tup` se vincula a toda la tupla porque una tupla se considera un único elemento compuesto. Para obtener los valores individuales de una tupla, podemos utilizar la concordancia de patrones para desestructurar un valor de tupla, así:

```rust
use debug::PrintTrait;
fn main() {
    let tup = (500, 6, true);

    let (x, y, z) = tup;

    if y == 6 {
        'y is six!'.print();
    }
}
```

Este programa crea primero una tupla y la asocia a la variable `tup`. A continuación, utiliza un patrón con `let` para tomar `tup` y convertirla en tres variables separadas, `x`, `y`, y `z`. Esto se llama _desestructuración_ porque divide la tupla en tres partes. Finalmente, el programa imprime `y es seis` ya que el valor de `y` es `6`.

También podemos declarar la tupla con valor y nombre al mismo tiempo.
Por ejemplo:

```rust
fn main() {
    let tup: (x: felt, y: felt) = (2,3);
}
```

#### El tipo Array

Otra forma de tener una colección de múltiples valores es con un _array_. A diferencia de
cada elemento de un array debe tener el mismo tipo. Puedes crear y utilizar métodos de array importando el trait `array::ArrayTrait`.

Una cosa importante a tener en cuenta es que los arrays son append-only. Esto significa que sólo puedes añadir elementos al final de un array.
Los arrays son, de hecho, colas cuyos valores no se pueden saltar ni modificar.
Esto tiene que ver con el hecho de que una vez que se escribe en un espacio de memoria, no se puede sobrescribir, sino sólo leer de él.

He aquí un ejemplo de creación de un array con 3 elementos:

```rust
use array::ArrayTrait;

fn main() {
    let mut a = ArrayTrait::new();
    a.append(0);
    a.append(1);
    a.append(2);
}
```

Es posible eliminar un elemento de la parte frontal de un array llamando al método `pop_front()`:

```rust

use option::OptionTrait;
use array::ArrayTrait;
use debug::PrintTrait;

fn main() {
    let mut a = ArrayTrait::new();
    a.append(10);
    a.append(1);
    a.append(2);

    let first_value = a.pop_front().unwrap();
    first_value.print();

}
```

El código anterior imprimirá `10` cuando eliminemos el primer elemento añadido.

Puedes pasar el tipo esperado de elementos dentro del array al instanciar el array así

```rust,
let mut arr = ArrayTrait::<u128>::new();
```

##### Acceso a los elementos de una matriz

Para acceder a los elementos de un array, puedes utilizar los métodos `get()` o `at()` que devuelven diferentes tipos. Utilizar `arr.at(index)` es equivalente a utilizar el operador de subíndice `arr[index]`.

La función `get` devuelve una `Option<Box<@T>>`, lo que significa que devuelve una opción a un tipo Box (el tipo smart-pointer de Cairo) que contiene una instantánea al elemento en el índice especificado si ese elemento existe en el array. Si el elemento no existe, `get` devuelve `None`. Este método es útil cuando esperas acceder a índices que pueden no estar dentro de los límites del array y quieres manejar tales casos con gracia sin pánicos. Las instantáneas se explicarán con más detalle en el capítulo [Referencias y Snapshots](ch03-02-references-and-snapshots.md).

La función `at`, por otro lado, devuelve directamente una instantánea al elemento en el índice especificado utilizando el operador `unbox()` para extraer el valor almacenado en una caja. Si el índice está fuera de los límites, se produce un error de pánico. Sólo debe utilizar at cuando desee que el programa entre en pánico si el índice proporcionado está fuera de los límites del array, lo que puede evitar comportamientos inesperados.

En resumen, usa `at` cuando quieras que el programa entre en pánico ante intentos de acceso fuera de los límites, y usa `get` cuando prefieras manejar estos casos con gracia sin entrar en pánico.

```rust
fn main() {
    let mut a = ArrayTrait::new();
    a.append(0);
    a.append(1);

    let first = *a.at(0_usize);
    let second = *a.at(1_usize);
}
```

En este ejemplo, la variable llamada `first` obtendrá el valor `0` porque es el valor del índice `0` del array. La variable llamada `second` obtendrá el valor `1` del índice `1` del array.

```rust
use array::ArrayTrait;
use box::BoxTrait;
fn main() -> u128 {
    let mut arr = ArrayTrait::<u128>::new();
    arr.append(100_u128);
    let length = arr.len();
    match arr.get(length - 1_usize) {
        Option::Some(x) => {
            *x.unbox()
        },
        Option::None(_) => {
            let mut data = ArrayTrait::new();
            data.append('out of bounds');
            panic(data)
        }
    } // returns 100
}
```

El ejemplo anterior muestra cómo podemos hacer una gestión de errores utilizando el método `get` en lugar del método `at`.

[control-de-flujo]: ch02-05-control-flow.html


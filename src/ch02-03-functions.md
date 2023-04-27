## Funciones

Las funciones son frecuentes en el código de Cairo. Ya has visto una de las funciones más importantes del lenguaje: la función `main`, que es el punto de entrada de muchos programas. También has visto la palabra clave `fn`, que te permite declarar nuevas funciones.

El código de Cairo usa *snake case* como estilo convencional para los nombres de funciones y variables, en el que todas las letras están en minúsculas y los guiones bajos separan las palabras.
Aquí hay un programa que contiene un ejemplo de definición de función:


```rust
use debug::PrintTrait;

fn another_function() {
    'Another function.'.print();
}

fn main() {
    'Hello, world!'.print();
    another_function();
}
```

Definimos una función en Cairo introduciendo `fn` seguido de un nombre de función y un
conjunto de paréntesis. Las llaves indican al compilador dónde empieza y termina el cuerpo de la función.

Podemos llamar a cualquier función que hayamos definido introduciendo su nombre seguido de un conjunto de paréntesis. Como `another_function` está definida en el programa, puede ser llamada desde dentro de la función `main`. Ten en cuenta que hemos definido "another_function" *antes* de la función `main` en el código fuente; también podríamos haberla definido después. A Cairo no le importa dónde definas tus funciones, sólo que estén definidas en algún lugar en un ámbito que pueda ser visto por quien las llama.

Empecemos un nuevo proyecto con Scarb llamado *functions* para explorar las funciones. Coloque el ejemplo `another_function` en *src/lib.cairo* y ejecútelo. Usted
Debería ver la siguiente salida:

```console
$ cairo-run src/lib.cairo
[DEBUG] Hello, world!                (raw: 5735816763073854953388147237921)
[DEBUG] Another function.            (raw: 22265147635379277118623944509513687592494)
```

Las líneas se ejecutan en el orden en que aparecen en la función `main`.
Primero se imprime el mensaje " Hello, world!", y luego se llama a `another_function` y se imprime su mensaje.

### Parámetros

Podemos definir funciones para que tengan *parámetros*, que son variables especiales que forman parte de la firma de una función. Cuando una función tiene parámetros, puede proporcionarle valores concretos para esos parámetros. Técnicamente, los valores
se llaman *argumentos*, pero en una conversación informal, la gente tiende a usar
las palabras *parámetro* y *argumento* indistintamente para las variables
en la definición de una función o los valores concretos pasados cuando se llama a una
función.

En esta versión de `another_function` añadimos un parámetro:

```rust
use debug::PrintTrait;

fn main() {
    another_function(5);
}

fn another_function(x: felt252) {
    x.print();
}
```

Intente ejecutar este programa; debería obtener la siguiente salida:

```console
$ cairo-run src/lib.cairo
[DEBUG]                                 (raw: 5)
```

La declaración de `another_function` tiene un parámetro llamado `x`. El tipo de
`x` se especifica como `felt252`. Cuando pasamos `5` a `another_function`, la función
`print()` muestra `5` en la consola.

En las firmas de función, *debes* declarar el tipo de cada parámetro. Esta es
una decisión deliberada en el diseño de Cairo: requerir anotaciones de tipo en las
significa que el compilador casi nunca necesita usarlas en otra parte del código
el código para averiguar a qué tipo se refiere. El compilador también es capaz de dar
mensajes de error más útiles si sabe qué tipos espera la función.

Cuando defina múltiples parámetros, separe las declaraciones de parámetros con
comas, así:

```rust
use debug::PrintTrait;

fn main() {
    another_function(5,6);
}

fn another_function(x: felt252, y:felt252) {
    x.print();
    y.print();
}
```

Este ejemplo crea una función llamada `another_function` con dos parámetros. El primer parámetro se llama `x` y es un `felt252`. El segundo se llama `y` y también es del tipo `felt252`. La función luego imprime el contenido del `x` y luego el contenido del `y`.

Intentemos ejecutar este código. Reemplaza el programa actualmente en el archivo *src/lib.cairo* de tu proyecto *functions* con el ejemplo anterior y ejecútalo usando `cairo-run src/lib.cairo`.

```console
$ cairo-run src/lib.cairo
[DEBUG]                                 (raw: 5)
[DEBUG]                                 (raw: 6)
```

Debido a que llamamos a la función con `5` como valor para `x` y `6` como valor para `y`, la salida del programa contiene esos valores.

### Sentencias y expresiones

Los cuerpos de las funciones están compuestos por una serie de sentencias que terminan opcionalmente en una expresión. Hasta ahora, las funciones que hemos cubierto no han incluido una expresión final, pero ya has visto una expresión como parte de una sentencia. Como Cairo es un lenguaje basado en expresiones, esta es una distinción importante que debemos entender. Otros lenguajes no tienen las mismas distinciones, así que veamos qué son las sentencias y expresiones y cómo sus diferencias afectan los cuerpos de las funciones.

* **Sentencias** son instrucciones que realizan alguna acción y no devuelven un valor.
* **Expresiones** se evalúan para producir un valor resultante. Veamos algunos ejemplos.

De hecho, ya hemos utilizado sentencias y expresiones. Crear una variable y asignarle un valor con la palabra clave `let` es una sentencia. En el Listado 3-1, `let y = 6;` es una sentencia.

```rust
fn main() {
    let y = 6;
}
```

<span class="caption">Listado 3-1: Una declaración de función `main` que contiene una sentencia</span>

Las definiciones de funciones también son sentencias; todo el ejemplo anterior es una sentencia en sí misma.

Las sentencias no devuelven valores. Por lo tanto, no se puede asignar una sentencia `let` a otra variable, como intenta hacer el siguiente código; se producirá un error:

```rust
fn main() {
    let x = (let y = 6);
}
```
Cuando ejecutes este programa, el error que obtendrás se verá así:
```console
$ cairo-run src/lib.cairo
error: Missing token TerminalRParen.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
             ^

error: Missing token TerminalSemicolon.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
             ^

error: Missing token TerminalSemicolon.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
                      ^

error: Skipped tokens. Expected: statement.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
```

La declaración `let y = 6` no devuelve un valor, por lo que no hay nada a lo que `x` pueda enlazar. Esto es diferente de lo que sucede en otros lenguajes, como C y Ruby, donde la asignación devuelve el valor de la asignación. En esos lenguajes, puedes escribir `x = y = 6` y tanto `x` como `y` tendrán el valor `6`; esto no es así en Cairo.

Las expresiones evalúan a un valor y componen la mayor parte del código que escribirás en Cairo. Considera una operación matemática, como `5 + 6`, que es una expresión que evalúa al valor `11`. Las expresiones pueden formar parte de las declaraciones: en el Listado 3-1, el `6` en la declaración `let y = 6;` es una expresión que evalúa al valor `6`. Llamar a una función es una expresión. Un bloque de ámbito nuevo creado con llaves es una expresión, por ejemplo:

```rust
use debug::PrintTrait;
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    y.print();
}
```

Esta expresión:
```rust
{
    let x = 3;
    x + 1
}
```
Este bloque de código, en este caso, se evalúa como 4. Ese valor se asigna a y como parte de la declaración let. Ten en cuenta que la línea x + 1 no tiene un punto y coma al final, lo que es diferente a la mayoría de las líneas que has visto hasta ahora. Las expresiones no incluyen un punto y coma al final. Si agregas un punto y coma al final de una expresión, la conviertes en una declaración, y en ese caso no se devolverá ningún valor. Tenlo en cuenta mientras exploras los valores de retorno de las funciones y las expresiones a continuación.

### Funciones con valores de retorno
Las funciones pueden devolver valores al código que las llama. No nombramos los valores de retorno, pero debemos declarar su tipo después de una flecha (`->`). En Cairo, el valor de retorno de la función es sinónimo del valor de la última expresión en el bloque del cuerpo de una función. Puede salir temprano de una función usando la palabra clave `return` y especificando un valor, pero la mayoría de las funciones devuelven la última expresión implícitamente. Aquí hay un ejemplo de una función que devuelve un valor:

```rust
use debug::PrintTrait;

fn five() -> u32 {
    5_u32
}

fn main() {
    let x = five();
    x.print();
}
```
No hay llamadas a funciones ni declaraciones `let` en la función `five`, solo el número `5` por sí mismo. Esa es una función perfectamente válida en Cairo. Observa que se especifica el tipo de retorno de la función como `-> u32`. Intenta ejecutar este código; la salida debería verse así:
```console
$ cairo-run src/lib.cairo
[DEBUG]                                 (raw: 5)
```
El `5` en `five` es el valor de retorno de la función, por eso el tipo de retorno es `u32`. Vamos a examinar esto con más detalle. Hay dos partes importantes: en primer lugar, la línea `let x = five();` muestra que estamos usando el valor de retorno de una función para inicializar una variable. Debido a que la función `five` devuelve un `5`, esa línea es lo mismo que:
```rust
let x = 5;
```
Segundo, la función `five` no tiene parámetros y define el tipo del valor de retorno, pero el cuerpo de la función es simplemente `5` sin un punto y coma porque es una expresión cuyo valor queremos retornar.

Veamos otro ejemplo:

```rust
use debug::PrintTrait;

fn main() {
    let x = plus_one(5_u32);

    x.print();
}

fn plus_one(x: u32) -> u32 {
    x + 1_u32
}
```
Al ejecutar este código se imprimirá `[DEBUG]                    (raw: 6)`. Pero si agregamos un punto y coma al final de la línea que contiene `x + 1`, cambiándola de una expresión a una declaración, obtendremos un error:

```rust
use debug::PrintTrait;

fn main() {
    let x = plus_one(5_u32);

    x.print();
}

fn plus_one(x: u32) -> u32 {
    x + 1_u32;
}
```

La compilación de este código produce un error, como se muestra a continuación:
```console
error: Unexpected return type. Expected: "core::integer::u32", found: "()".
```
El mensaje principal de error, `Unexpected return type`, revela el problema principal con este código. La definición de la función `plus_one` dice que devolverá un `u32`, pero las sentencias no se evalúan a un valor, lo cual se expresa por `()`, el tipo unit. Por lo tanto, no se devuelve nada, lo que contradice la definición de la función y resulta en un error.
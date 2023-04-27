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

### Statements and Expressions

Function bodies are made up of a series of statements optionally ending in an
expression. So far, the functions we’ve covered haven’t included an ending
expression, but you have seen an expression as part of a statement. Because
Cairo is an expression-based language, this is an important distinction to
understand. Other languages don’t have the same distinctions, so let’s look at
what statements and expressions are and how their differences affect the bodies
of functions.

* **Statements** are instructions that perform some action and do not return
  a value.
* **Expressions** evaluate to a resultant value. Let’s look at some examples.

We’ve actually already used statements and expressions. Creating a variable and
assigning a value to it with the `let` keyword is a statement. In Listing 3-1,
`let y = 6;` is a statement.

```rust
fn main() {
    let y = 6;
}
```

<span class="caption">Listing 3-1: A `main` function declaration containing one statement</span>

Function definitions are also statements; the entire preceding example is a
statement in itself.

Statements do not return values. Therefore, you can’t assign a `let` statement
to another variable, as the following code tries to do; you’ll get an error:

```rust
fn main() {
    let x = (let y = 6);
}
```
When you run this program, the error you’ll get looks like this:
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

The `let y = 6` statement does not return a value, so there isn’t anything for
`x` to bind to. This is different from what happens in other languages, such as
C and Ruby, where the assignment returns the value of the assignment. In those
languages, you can write `x = y = 6` and have both `x` and `y` have the value
`6`; that is not the case in Cairo.

Expressions evaluate to a value and make up most of the rest of the code that
you’ll write in Cairo. Consider a math operation, such as `5 + 6`, which is an
expression that evaluates to the value `11`. Expressions can be part of
statements: in Listing 3-1, the `6` in the statement `let y = 6;` is an
expression that evaluates to the value `6`. Calling a function is an
expression. A new scope block created with
curly brackets is an expression, for example:


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

This expression:
```rust
{
    let x = 3;
    x + 1
}
```
is a block that, in this case, evaluates to `4`. That value gets bound to `y`
as part of the `let` statement. Note that the `x + 1` line doesn’t have a
semicolon at the end, which is unlike most of the lines you’ve seen so far.
Expressions do not include ending semicolons. If you add a semicolon to the end
of an expression, you turn it into a statement, and it will then not return a
value. Keep this in mind as you explore function return values and expressions
next.
### Functions with Return Values
Functions can return values to the code that calls them. We don’t name return
values, but we must declare their type after an arrow (`->`). In Cairo, the
return value of the function is synonymous with the value of the final
expression in the block of the body of a function. You can return early from a
function by using the `return` keyword and specifying a value, but most
functions return the last expression implicitly. Here’s an example of a
function that returns a value:

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
There are no function calls, or even `let` statements in the `five`
function—just the number `5` by itself. That’s a perfectly valid function in
Cairo. Note that the function’s return type is specified too, as `-> u32`. Try
running this code; the output should look like this:
```console
$ cairo-run src/lib.cairo
[DEBUG]                                 (raw: 5)
```
The `5` in `five` is the function’s return value, which is why the return type
is `u32`. Let’s examine this in more detail. There are two important bits:
first, the line `let x = five();` shows that we’re using the return value of a
function to initialize a variable. Because the function `five` returns a `5`,
that line is the same as the following:
```rust
let x = 5;
```
Second, the `five` function has no parameters and defines the type of the
return value, but the body of the function is a lonely `5` with no semicolon
because it’s an expression whose value we want to return.
Let’s look at another example:

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
Running this code will print `[DEBUG]                    (raw: 6)`. But if we place a
semicolon at the end of the line containing `x + 1`, changing it from an
expression to a statement, we’ll get an error:

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

Compiling this code produces an error, as follows:
```console
error: Unexpected return type. Expected: "core::integer::u32", found: "()".
```
The main error message, `Unexpected return type`, reveals the core issue with this
code. The definition of the function `plus_one` says that it will return an
`u32`, but statements don’t evaluate to a value, which is expressed by `()`,
the unit type. Therefore, nothing is returned, which contradicts the function
definition and results in an error.
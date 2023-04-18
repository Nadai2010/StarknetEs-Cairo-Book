## Variables y mutabilidad

Cairo usa un modelo de memoria inmutable, lo que significa que una vez que se escribe en una celda de memoria, no puede ser sobrescrita sino sólo leída. Para reflejar este modelo de memoria inmutable, las variables en Cairo son inmutables por defecto.
Sin embargo, el lenguaje abstrae este modelo y te da la opción de hacer tus
variables mutables. Exploremos cómo y por qué Cairo impone la inmutabilidad, y cómo
puedes hacer tus variables mutables.

Cuando una variable es inmutable, una vez que un valor está ligado a un nombre, no puedes cambiar ese valor. Para ilustrar esto, genera un nuevo proyecto llamado _variables_ en tu directorio _cairo_projects_ usando `scarb new variables`.

A continuación, en su nuevo directorio _variables_, abra _src/lib.cairo_ y sustituya su código por el siguiente, que todavía no compilará:

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;
fn main() {
    let x = 5;
    x.print();
    x = 6;
    x.print();
}
```

Guarde y ejecute el programa utilizando `cairo-run src/lib.cairo`. Debería recibir un mensaje de error relativo a un error de inmutabilidad, como se muestra en esta salida:

```console
error: Cannot assign to an immutable variable.
 --> lib.cairo:5:5
    x = 6;
    ^***^

Error: failed to compile: src/lib.cairo
```

Este ejemplo muestra cómo el compilador te ayuda a encontrar errores en tus programas.
Los errores del compilador pueden ser frustrantes, pero en realidad sólo significan que su programa 
todavía no está haciendo con seguridad lo que usted quiere que haga; ¡no significan que usted no sea un buen programador! Los Caironautas experimentados siguen teniendo errores de compilador.

Recibiste el mensaje de error `Cannot assign to an immutable variable.` porque intentaste asignar un segundo valor a la variable inmutable `x`.

Es importante que obtengamos errores en tiempo de compilación cuando intentamos cambiar un valor designado como inmutable porque esta situación específica puede conducir a errores. Si una parte de nuestro código opera bajo la suposición de que un valor nunca cambiará y otra parte de nuestro código cambia ese valor, es posible
que la primera parte del código no haga lo que fue diseñada para hacer. La causa de este tipo de error puede ser difícil de rastrear después de los hechos, especialmente cuando la segunda parte del código cambia el valor sólo _a veces_. El compilador Cairo garantiza que cuando dices que un valor no cambiará, realmente no cambiará, por lo que no tiene que hacer un seguimiento. Su código es así más fácil de razonar.

Pero la mutabilidad puede ser muy útil, y puede hacer que el código sea más cómodo de escribir.
Aunque las variables son inmutables por defecto, puedes hacerlas mutables añadiendo `mut` delante del nombre de la variable. Añadir `mut` también transmite intención a los futuros lectores del código indicando que otras partes del código cambiarán el valor de esta variable.

Sin embargo, puede que en este punto te estés preguntando qué ocurre exactamente cuando una variable es declarada como `mut`, ya que previamente mencionamos que la memoria de Cairo es inmutable.
La respuesta es que la memoria de Cairo es inmutable, pero la dirección de memoria a la que apunta la variable puede ser cambiada. Al examinar el código ensamblador de bajo nivel de Cairo, queda claro que la mutación de variables se implementa como azúcar sintáctico, que traduce las operaciones de mutación en una serie de pasos equivalentes al shadowing de variables. La única diferencia es que en el nivel la variable no se vuelve a declarar, por lo que su tipo no puede cambiar.

Por ejemplo, cambiemos _src/lib.cairo_ por lo siguiente:

<span class="filename">Filename: src/lib.cairo</span>

```rust
use debug::PrintTrait;
fn main() {
    let mut x = 5;
    x.print();
    x = 6;
    x.print();
}
```

Cuando ejecutamos el programa ahora, obtenemos esto:

```console
❯ cairo-run src/lib.cairo
[DEBUG]	                              	(raw: 5)

[DEBUG]	                              	(raw: 6)

Run completed successfully, returning []
```

Se nos permite cambiar el valor ligado a `x` de `5` a `6` cuando se usa `mut`. En última instancia, la decisión de utilizar la mutabilidad o no es suya y depende de lo que usted piensa que es más claro en esa situación particular.

##### Constantes

Al igual que las variables inmutables, las _constantes_ son valores que están vinculados a un nombre y no se les permite cambiar, pero hay algunas diferencias entre las constantes y las variables.

En primer lugar, no se permite el uso de `mut` con las constantes. Las constantes no son solo inmutables por defecto, sino que siempre son inmutables. Se declaran constantes usando la palabra clave `const` en lugar de la palabra clave `let`, y el tipo de valor _debe_ ser anotado. Cubriremos los tipos y las anotaciones de tipo en la próxima sección, [“Tipos de datos”][tipos-de-datos]<!-- ignore -->, así que no se preocupe por los detalles por ahora. Solo sepa que siempre debe anotar el tipo.

Las constantes solo se pueden declarar en el ámbito global, lo que las hace útiles para valores que muchas partes del código deben conocer.

La última diferencia es que las constantes solo pueden ser asignadas a una expresión constante, no al resultado de un valor que solo se podría calcular en tiempo de ejecución. Actualmente, solo se admiten constantes literales.

Aquí hay un ejemplo de declaración de constante:


```rust
const ONE_HOUR_IN_SECONDS: u32 = 3600_u32;
```

La convención de nomenclatura de Cairo para las constantes es usar todas las mayúsculas con guiones bajos entre palabras.

La convención de nombres de constantes en Cairo es utilizar todo en mayúsculas con guiones bajos entre palabras.

Las constantes son válidas durante todo el tiempo que se ejecuta un programa, dentro del ámbito en el que fueron declaradas. Esta propiedad hace que las constantes sean útiles para los valores en el dominio de su aplicación que varias partes del programa podrían necesitar conocer, como el número máximo de puntos que cualquier jugador de un juego puede ganar o la velocidad de la luz.

Nombrar los valores codificados en duro utilizados en todo el programa como constantes es útil para transmitir el significado de ese valor a los futuros mantenedores del código. También ayuda a tener solo un lugar en su código donde tendría que cambiar si el valor codificado en duro necesitara ser actualizado en el futuro.

### Shadowing

La sombra de una variable se refiere a la declaración de una nueva variable con el mismo nombre que una variable anterior. Los caironautas dicen que la primera variable está _sombreada_ por la segunda, lo que significa que el compilador verá la segunda variable cuando use el nombre de la variable. En efecto, la segunda variable oculta la primera, tomando cualquier uso del nombre de la variable para sí misma hasta que ella misma sea sombreada o que el ámbito termine. Podemos sombrear una variable usando el mismo nombre de la variable y repitiendo el uso de la palabra clave `let` de la siguiente manera:

<span class="filename">Filename: src/lib.cairo</span>

```swift
use debug::PrintTrait;
fn main() {
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        'Inner scope x value is:'.print();
        x.print()
    }
    'Outer scope x value is:'.print();
    x.print();
}
```

Este programa primero asigna un valor de `5` a `x`. Luego crea una nueva variable `x` repitiendo `let x =`, tomando el valor original y sumando `1`, por lo que el valor de `x` es ahora `6`. Luego, dentro de un ámbito interno creado con llaves, la tercera instrucción `let` también sombrea `x` y crea una nueva variable, multiplicando el valor anterior por `2` para darle a `x` un valor de `12`. Cuando ese ámbito termina, la sombra interna termina y `x` vuelve a ser `6`. Al ejecutar este programa, se mostrará lo siguiente:

```console
cairo-run src/lib.cairo
[DEBUG]	Inner scope x value is:        	(raw: 7033328135641142205392067879065573688897582790068499258)

[DEBUG]
                                      	(raw: 12)

[DEBUG]	Outer scope x value is:        	(raw: 7610641743409771490723378239576163509623951327599620922)

[DEBUG]	                              	(raw: 6)

Run completed successfully, returning []
```
El sombreado es diferente de marcar una variable como `mut`, porque obtendremos un error en tiempo de compilación si intentamos reasignar a esta variable sin usar la palabra clave `let`. Al usar `let`, podemos realizar algunas transformaciones en un valor pero hacer que la variable sea inmutable después de que se hayan completado esas transformaciones.

Otra diferencia entre `mut` y el sombreado es que al usar la palabra clave `let` nuevamente, estamos creando efectivamente una nueva variable, lo que nos permite cambiar el tipo del valor mientras reutilizamos el mismo nombre. Como se mencionó antes, el sombreado de variables y las variables mutables son equivalentes a un nivel más bajo. La única diferencia es que al sombrear una variable, el compilador no se quejará si cambia su tipo. Por ejemplo, digamos que nuestro programa realiza una conversión de tipo entre los tipos `u64` y `felt252`.

```rust
use debug::PrintTrait;
use traits::Into;
fn main() {
    let x = 2_u64;
    x.print();
    let x = x.into(); // converts x to a felt.
    x.print()
}
```

LEl primer variable `x` tiene un tipo `u64`, mientras que la segunda variable `x` tiene un tipo `felt252`.
Por lo tanto, el shadowing nos ahorra tener que inventar diferentes nombres, como `x_u64` y `x_felt252`; en su lugar, podemos reutilizar el nombre más simple `x`. Sin embargo, si intentamos usar `mut` para esto, como se muestra aquí, obtendremos un error en tiempo de compilación:

```rust
use debug::PrintTrait;
use traits::Into;
fn main() {
    let mut x = 2_u64;
    x.print();
    x = x.into();
    x.print()
}
```

El error dice que se esperaba un `u64` (el tipo original) pero se obtuvo un tipo diferente:

```console
❯ cairo-run src/lib.cairo
error: Unexpected argument type. Expected: "core::integer::u64", found: "core::felt252".
 --> lib.cairo:6:9
    x = x.into();
        ^******^

Error: failed to compile: src/lib.cairo
```

Ahora que hemos explorado cómo funcionan las variables, veamos otros tipos de datos que pueden tener.

[data-types]: ch03-02-data-types.html#data-types

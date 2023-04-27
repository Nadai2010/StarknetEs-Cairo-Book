# Cómo escribir Test

## La Anatomía de una Función de Testing

Las pruebas son funciones en Cairo que verifican que el código no relacionado con las pruebas está funcionando de la manera esperada. Los cuerpos de las funciones de prueba típicamente realizan estas tres acciones:

- Configuran cualquier dato o estado necesario.
- Ejecutan el código que se desea probar.
- Verifican que los resultados sean los esperados.

Veamos las características específicas que Cairo proporciona para escribir pruebas que realizan estas acciones, que incluyen el atributo `test`, la función `assert` y el atributo `should_panic`.

### La anatomía de una función de prueba

En su forma más simple, una prueba en Cairo es una función que está anotada con el atributo `test`. Los atributos son metadatos sobre piezas de código en Cairo; un ejemplo es el atributo `derive` que usamos con estructuras en el capítulo 4. Para convertir una función en una función de prueba, agrega `#[test]` en la línea antes de `fn`. Cuando se ejecutan las pruebas con el comando `cairo-test`, Cairo construye un binario de ejecución de pruebas que ejecuta las funciones anotadas y reporta si cada función de prueba pasa o falla.

Creemos un nuevo proyecto llamado `adder` que sumará dos números usando Scarb con el comando `scarb new adder`: "

```shell
adder
├── cairo_project.toml
├── Scarb.toml
└── src
    └── lib.cairo
```
<!-- TODO: remove when Scarb test work -->

> Nota: Aquí notarás un archivo `cairo_project.toml`.
> Este es el archivo de configuración para proyectos Cairo "vanilla" (es decir, no administrados por Scarb),
> que se requiere para ejecutar el comando `cairo-test .` para ejecutar el código del crate.
> Es necesario hasta que Scarb implemente esta característica. El contenido del archivo es:
>
> ```toml
> [crate_roots]
> adder = "src"
> ```
>
> e indica que el crate llamado "adder" se encuentra en el directorio `src`.

En _lib.cairo_, agreguemos una primera prueba, como se muestra en el Listado 8-1.

<span class="filename">Nombre de archivo: lib.cairo</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert(result == 4, 'result is not 4');
    }
}
```

Listado 8-1: Un módulo y función de prueba

Por ahora, ignoraremos las dos primeras líneas y nos centraremos en la función. Observa la anotación `#[test]`: este atributo indica que esta es una función de prueba, por lo que el runner de pruebas sabe que debe tratar esta función como una prueba. También podríamos tener funciones que no son de prueba en el módulo de pruebas para ayudar a configurar escenarios comunes o realizar operaciones comunes, por lo que siempre debemos indicar qué funciones son pruebas.

El cuerpo de la función de ejemplo utiliza la función `assert`, que comprueba que el resultado de sumar 2 y 2 es igual a 4. Esta afirmación sirve como ejemplo del formato de una prueba típica. Ejecutémoslo para ver que esta prueba pasa.

El comando `cairo-test .` ejecuta todas las pruebas en nuestro proyecto, como se muestra en el Listado 8-2.

```shell
$ cairo-test .
running 1 tests
test adder::lib::tests::it_works ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

"Listado 8-2: La salida al ejecutar una prueba

`cairo-test` compiló y ejecutó la prueba. Vemos la línea `running 1 tests`. La siguiente línea muestra el nombre de la función de prueba generada, llamada `it_works`, y que el resultado de ejecutar esa prueba es `ok`. El resumen general `test result: ok.` significa que todas las pruebas pasaron, y la porción que lee `1 passed; 0 failed` totaliza el número de pruebas que pasaron o fallaron.

Es posible marcar una prueba como ignorada para que no se ejecute en una instancia particular; cubriremos eso en la sección [Ignorando algunas pruebas a menos que se soliciten específicamente](#ignoring-some-tests-unless-specifically-requested) más adelante en este capítulo. Debido a que no hemos hecho eso aquí, el resumen muestra `0 ignoradas`. También podemos pasar un argumento al comando `cairo-test` para ejecutar solo una prueba cuyo nombre coincida con una cadena; esto se llama filtrado y lo cubriremos en la sección [Ejecución de pruebas individuales](#running-single-tests). Tampoco hemos filtrado las pruebas que se ejecutan, por lo que el final del resumen muestra `0 filtradas`.

Comencemos a personalizar la prueba según nuestras necesidades. Primero, cambie el nombre de la función `it_works` a un nombre diferente, como `exploration`, así:

<span class="filename">Nombre de archivo: lib.cairo</span>"

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        let result = 2 + 2;
        assert(result == 4, 'result is not 4');
    }
}
```

Luego vuelva a ejecutar `cairo-test  -- --path src`. La salida ahora muestra `exploration` en lugar de `it_works`:

```shell
$ cairo-test .
running 1 tests
test adder::lib::tests::exploration ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

Ahora agregaremos otra prueba, ¡pero esta vez haremos una prueba que falla! Las pruebas fallan cuando algo en la función de prueba causa un pánico. Cada prueba se ejecuta en un hilo nuevo y cuando el hilo principal ve que un hilo de prueba ha muerto, la prueba se marca como fallida. Agregue la nueva prueba como una función llamada `another`, de modo que su archivo _src/lib.cairo_ se vea como en el Listado 8-3.

```rust
#[cfg(test)]
mod tests{
    #[test]
    fn another() {
        let result = 2 + 2;
        assert(result == 6, 'Make this test fail');
    }
}
```

Lista 8-3: Agregando una segunda prueba que fallará.

```shell
$ cairo-test .
running 2 tests
test adder::lib::tests::exploration ... ok
test adder::lib::tests::another ... fail
failures:
    adder::lib::tests::another - panicked with [1725643816656041371866211894343434536761780588 ('Make this test fail'), ].
Error: test result: FAILED. 1 passed; 1 failed; 0 ignored
```

Listing 8-4: Resultados de las pruebas cuando una pasa y otra falla

En lugar de `ok`, la línea `adder::lib::tests::another` muestra `fail`. Aparece una nueva sección entre los resultados individuales y el resumen. Muestra la razón detallada de cada falla de prueba. En este caso, obtenemos los detalles de que `another` falló porque falló con un pánico con `[1725643816656041371866211894343434536761780588 ('Make this test fail'), ]` en el archivo _src/lib.cairo_.

La línea de resumen se muestra al final: en general, nuestro resultado de prueba es `FAILED`. Tuvimos una prueba que pasó y otra que falló.

Ahora que ha visto cómo son los resultados de las pruebas en diferentes escenarios, veamos algunas funciones que son útiles en las pruebas.

## Verificar resultados con la función assert

La función `assert`, proporcionada por Cairo, es útil cuando desea asegurarse de que alguna condición en una prueba se evalúe como verdadera. Le damos a la función `assert` un primer argumento que se evalúa como un valor booleano. Si el valor es `true`, no sucede nada y la prueba pasa. Si el valor es `false`, la función `assert` llama a `panic()` para hacer que la prueba falle con un mensaje que definimos como segundo argumento de la función `assert`. Usar la función `assert` nos ayuda a verificar que nuestro código funciona de la manera que pretendemos.

En [Capítulo 4, Lista 5-15](ch04-03-method-syntax.md#multiple-impl-blocks), usamos una estructura `Rectangle` y un método `can_hold`, que se repiten aquí en la Lista 8-5. Colocaremos este código en el archivo _src/lib.cairo_, luego escribiremos algunas pruebas para él usando la función `assert`.

<span class="filename">Nombre de archivo: lib.cairo</span>

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

Lista 8-5: Uso de la estructura `Rectangle` y su método `can_hold` del Capítulo 5

El método `can_hold` devuelve un valor booleano, lo que significa que es un caso de uso perfecto para la función `assert`. En el Listado 8-6, escribimos una prueba que ejerce el método `can_hold` creando una instancia de `Rectangle` que tiene un ancho de `8_u64` y una altura de `7_u64` y asegurando que puede contener otra instancia de `Rectangle` que tiene un ancho de `5_u64` y una altura de `1_u64`.

<span class="filename">Nombre de archivo: lib.cairo</span>

```rust
#[cfg(test)]
mod tests {
    use super::Rectangle;
    use super::RectangleTrait;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            height: 7_u64,
            width: 8_u64,
        };
        let smaller = Rectangle {
            height: 1_u64,
            width: 5_u64,
        };

        assert(larger.can_hold(@smaller), 'rectangle cannot hold');
    }
}
```

Lista 8-6: Un test para `can_hold` que verifica si un rectángulo más grande realmente puede contener un rectángulo más pequeño

Note que hemos agregado dos nuevas líneas dentro del módulo de pruebas: `use super::Rectangle;` y `use super::RectangleTrait;`. El módulo de pruebas es un módulo regular que sigue las reglas normales de visibilidad. Debido a que el módulo de pruebas es un módulo interno, necesitamos traer el código bajo prueba en el módulo externo al ámbito del módulo interno.

Hemos nombrado nuestro test `larger_can_hold_smaller`, y hemos creado los dos instancias de `Rectangle` que necesitamos. Luego llamamos a la función assert y le pasamos el resultado de llamar a `larger.can_hold(@smaller)`. Esta expresión se supone que devuelve `true`, por lo que nuestra prueba debería pasar. ¡Descubramoslo!

```shell
$ cairo-test .
running 1 tests
test adder::lib::tests::larger_can_hold_smaller ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

¡Pasó la prueba! Ahora agreguemos otra prueba, esta vez afirmamos que un rectángulo más pequeño no puede contener un rectángulo más grande:

<span class="filename">Nombre de archivo: lib.cairo</span>

```rust
#[cfg(test)]
mod tests {
    use super::Rectangle;
    use super::RectangleTrait;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            height: 7_u64,
            width: 8_u64,
        };
        let smaller = Rectangle {
            height: 1_u64,
            width: 5_u64,
        };

        assert(!smaller.can_hold(@larger), 'rectangle cannot hold');
    }
}
```

Como el resultado correcto de la función `can_hold` en este caso es `false`, debemos negar ese resultado antes de pasarlo a la función `assert`. Como resultado, nuestro test pasará si `can_hold` devuelve false:

```shell
$ cairo-test .
    running 2 tests
    test adder::lib::tests::smaller_cannot_hold_larger ... ok
    test adder::lib::tests::larger_can_hold_smaller ... ok
    test result: ok. 2 passed; 0 failed; 0 ignored; 0 filtered out;
```

¡Dos pruebas que pasan! Ahora veamos qué sucede con los resultados de nuestras pruebas cuando introducimos un error en nuestro código. Cambiaremos la implementación del método `can_hold` reemplazando el signo mayor que (`>`) por un signo menor que (`<`) cuando compara los anchos:

```rust
// --snip--
impl RectangleImpl of RectangleTrait {
    fn can_hold(self: @Rectangle, other: @Rectangle) -> bool {
        *self.width < *other.width & *self.height > *other.height
    }
}
```

Ejecutando los test ahora produce lo siguiente:

```shell
$ cairo-test .
running 2 tests
test adder::lib::tests::smaller_cannot_hold_larger ... ok
test adder::lib::tests::larger_can_hold_smaller ... fail
failures:
   adder::lib::tests::larger_can_hold_smaller - panicked with [167190012635530104759003347567405866263038433127524 ('rectangle cannot hold'), ].

Error: test result: FAILED. 1 passed; 1 failed; 0 ignored
```

Nuestros tests detectaron el error! Debido a que `larger.width` es `8_u64` y `smaller.width` es `5_u64`, la comparación de anchuras en `can_hold` ahora devuelve `false`: `8_u64` no es menor que `5_u64`.

## Comprobando los pánicos con `should_panic`

Además de verificar los valores de retorno, es importante verificar que nuestro código maneje las condiciones de error como esperamos. Por ejemplo, consideremos el tipo Guess en el Listing 8-8. Otro código que usa `Guess` depende de la garantía de que las instancias de `Guess` contengan solo valores entre `1_u64` y `100_u64`. Podemos escribir un test que asegure que al intentar crear una instancia de `Guess` con un valor fuera de ese rango, el programa entra en pánico.

Lo hacemos agregando el atributo `should_panic` a nuestra función de prueba. La prueba pasa si el código dentro de la función entra en pánico; la prueba falla si el código dentro de la función no entra en pánico.

El Listing 8-8 muestra una prueba que verifica que las condiciones de error de `GuessTrait::new` ocurren cuando esperamos que sucedan.

<span class="filename">Nombre de archivo: lib.cairo</span>

```rust
use array::ArrayTrait;

#[derive(Copy, Drop)]
struct Guess {
    value: u64,
}

trait GuessTrait {
    fn new(value: u64) -> Guess;
}

impl GuessImpl of GuessTrait {
    fn new(value: u64) -> Guess {
        if value < 1_u64 | value > 100 {
            let mut data = ArrayTrait::new();
            data.append('Guess must be >= 1 and <= 100');
            panic(data);
        }
        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::Guess;
    use super::GuessTrait;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        GuessTrait::new(200_u64);
    }
}
```

Listing 8-8: Probando que una condición causará un pánico

Colocamos el atributo `#[should_panic]` después del atributo `#[test]` y antes de la función de prueba a la que se aplica. Veamos el resultado cuando esta prueba pasa: 

<span class="filename">Nombre de archivo: lib.cairo</span>

```shell
$ cairo-test .
running 1 tests
test adder::lib::tests::greater_than_100 ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

Looks good! Now let’s introduce a bug in our code by removing the condition that the new function will panic if the value is greater than `100_u64`:

```rust
// --snip--
impl GuessImpl of GuessTrait {
    fn new(value: u64) -> Guess {
        if value < 1_u64 {
            let mut data = ArrayTrait::new();
            data.append('Guess must be >= 1 and <= 100');
            panic(data);
        }

        Guess { value, }
    }
}
```

Cuando ejecutamos la prueba en el Listado 8-8, fallará:

```shell
$ cairo-test .
running 1 tests
test adder::lib::tests::greater_than_100 ... fail
failures:
   adder::lib::tests::greater_than_100 - expected panic but finished successfully.
Error: test result: FAILED. 0 passed; 1 failed; 0 ignored
```

En este caso, no obtenemos un mensaje muy útil, pero cuando miramos la función de prueba, vemos que está anotada con `#[should_panic]`. La falla que obtuvimos significa que el código en la función de prueba no causó un pánico.

Las pruebas que usan `should_panic` pueden ser imprecisas. Una prueba con `should_panic` pasaría incluso si la prueba produce un pánico por una razón diferente a la que esperábamos. Para hacer que las pruebas con `should_panic` sean más precisas, podemos agregar un parámetro opcional `expected` al atributo `should_panic`. El sistema de pruebas se asegurará de que el mensaje de error contenga el texto proporcionado. Por ejemplo, considere el código modificado para `Guess` en el Listado 8-9, donde la nueva función genera un pánico con mensajes diferentes dependiendo de si el valor es demasiado pequeño o demasiado grande.  

<span class="filename">Nombre del archivo: lib.cairo</span>

```rust
// --snip--
impl GuessImpl of GuessTrait {
    fn new(value: u64) -> Guess {
        if value < 1_u64 {
            let mut data = ArrayTrait::new();
            data.append('Guess must be >= 1');
            panic(data);
        } else if value > 100_u64 {
            let mut data = ArrayTrait::new();
            data.append('Guess must be <= 100');
            panic(data);
        }

        Guess { value, }
    }
}

#[cfg(test)]
mod tests {
    use super::Guess;
    use super::GuessTrait;

    #[test]
    #[should_panic(expected: ('Guess must be <= 100', ))]
    fn greater_than_100() {
        GuessTrait::new(200_u64);
    }
}
```

Listado 8-9: Prueba para una excepción con un mensaje de excepción que contiene la cadena del mensaje de error

Esta prueba pasará porque el valor que ponemos en el parámetro esperado del atributo `should_panic` es la matriz de cadenas del mensaje con el que la función `Guess::new` genera la excepción. Necesitamos especificar el mensaje completo de la excepción que esperamos.

Para ver qué sucede cuando una prueba `should_panic` con un mensaje esperado falla, introduzcamos de nuevo un error en nuestro código cambiando los cuerpos de los bloques if `value < 1_u64` y else if `value > 100_u64`:

```rust
if value < 1_u64 {
    let mut data = ArrayTrait::new();
    data.append('Guess must be <= 100');
    panic(data);
} else if value > 100_u64 {
    let mut data = ArrayTrait::new();
    data.append('Guess must be >= 1');
    panic(data);
}
```

Esta vez, cuando ejecutamos la prueba `should_panic`, fallará:

```shell
$ cairo-test .
running 1 tests
test adder::lib::tests::greater_than_100 ... fail
failures:
   adder::lib::tests::greater_than_100 - panicked with [6224920189561486601619856539731839409791025 ('Guess must be >= 1'), ].

Error: test result: FAILED. 0 passed; 1 failed; 0 ignored
```

El mensaje de fallo indica que este test realmente causó un pánico como esperábamos, pero el mensaje de pánico no incluyó la cadena esperada. El mensaje de pánico que obtuvimos en este caso fue `Guess must be >= 1`. ¡Ahora podemos comenzar a descubrir dónde está nuestro error!

## Ejecución de pruebas individuales

A veces, ejecutar un conjunto completo de pruebas puede llevar mucho tiempo. Si está trabajando en código en un área particular, es posible que desee ejecutar solo las pruebas relacionadas con ese código. Puede elegir qué pruebas ejecutar pasando el nombre de la prueba que desea ejecutar como argumento a `cairo-test`.

Para demostrar cómo ejecutar una sola prueba, primero crearemos dos funciones de prueba, como se muestra en el Listado 8-10, y elegiremos cuáles ejecutar.

<span class="filename">Nombre de archivo: src/lib.cairo</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn add_two_and_two() {
        let result = 2 + 2;
        assert(result == 4, 'result is not 4');
    }

    #[test]
    fn add_three_and_two() {
        let result = 3 + 2;
        assert(result == 5, 'result is not 5');
    }
}
```

Listado 8-10: Dos pruebas con dos nombres diferentes

Podemos pasar el nombre de cualquier función de prueba a `cairo-test` para ejecutar solo esa prueba usando la bandera `-f`:

```shell
$ cairo-test . -f add_two_and_two
running 1 tests
test adder::lib::tests::add_two_and_two ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 1 filtered out;
```

Solo se ejecutó la prueba con el nombre `add_two_and_two`; la otra prueba no coincidía con ese nombre. La salida de la prueba nos indica que tuvimos una prueba más que no se ejecutó al mostrar "1 filtrado" al final.

También podemos especificar parte del nombre de una prueba y se ejecutarán todas las pruebas cuyo nombre contenga ese valor.

## Ignorar algunas pruebas a menos que se soliciten específicamente

A veces, algunas pruebas específicas pueden ser muy lentas de ejecutar, por lo que es posible que desee excluirlos durante la mayoría de las ejecuciones de `cairo-test`. En lugar de enumerar como argumentos todas las pruebas que desea ejecutar, puede anotar las pruebas que consumen mucho tiempo utilizando el atributo `ignore` para excluirlos, como se muestra aquí:

<span class="filename">Nombre de archivo: src/lib.cairo</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert(result == 4, 'result is not 4');
    }

    #[test]
    #[ignore]
    fn expensive_test() {
        // code that takes an hour to run
    }
}
```

Después de `#[test]` agregamos la línea `#[ignore]` al test que queremos excluir. Ahora, cuando ejecutamos nuestros tests, `it_works` se ejecuta pero `expensive_test` no lo hace:

```shell
$ cairo-test .
running 2 tests
test adder::lib::tests::expensive_test ... ignored
test adder::lib::tests::it_works ... ok
test result: ok. 1 passed; 0 failed; 1 ignored; 0 filtered out;
```

La función `expensive_test` está listada como ignorada.

Cuando esté en un punto en el que tenga sentido verificar los resultados de las pruebas ignoradas y tenga tiempo para esperar los resultados, puede ejecutar `cairo-test --include-ignored` para ejecutar todas las pruebas, ya sea que estén ignoradas o no.

# La construcción de control de flujo Match

Cairo tiene una construcción de control de flujo extremadamente poderosa llamada `match` que te permite comparar un valor con una serie de patrones y luego ejecutar código basado en el patrón que coincide. Los patrones pueden estar compuestos por valores literales, nombres de variables, comodines y muchas otras cosas. El poder de `match` proviene de la expresividad de los patrones y del hecho de que el compilador confirma que se manejan todos los casos posibles.

Piensa en una expresión `match` como una máquina clasificadora de monedas: las monedas se deslizan por una pista con agujeros de diferentes tamaños a lo largo de ella, y cada moneda cae por el primer agujero que encuentra en el que encaja. De la misma manera, los valores pasan por cada patrón en un `match`, y en el primer patrón en el que el valor "encaja", el valor cae en el bloque de código asociado para ser utilizado durante la ejecución.

Hablando de monedas, ¡usemoslas como ejemplo con `match`! Podemos escribir una función que toma una moneda de EE. UU. desconocida y, de manera similar a la máquina de contar, determina qué moneda es y devuelve su valor en centavos, como se muestra en el Listado 5-3.

```rust
enum Coin {
    Penny: (),
    Nickel: (),
    Dime: (),
    Quarter: (),
}

fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => 1,
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(_)=> 25,
    }
}
```

Listado 5-3: Un enum y una expresión `match` que tiene las variantes del enum como sus patrones.

Desglosemos el `match` en la función `value_in_cents`. Primero enumeramos la palabra clave `match` seguida de una expresión, que en este caso es el valor `coin`. Esto parece muy similar a una expresión condicional utilizada con `if`, pero hay una gran diferencia: con `if`, la condición debe evaluarse a un valor booleano, pero aquí puede ser de cualquier tipo. El tipo de moneda en este ejemplo es el enum `Coin` que definimos en la primera línea.

A continuación, están los brazos del `match`. Un brazo tiene dos partes: un patrón y algún código. El primer brazo aquí tiene un patrón que es el valor `Coin::Penny(_)` y luego el operador `=>` que separa el patrón y el código a ejecutar. El código en este caso es simplemente el valor `1`. Cada brazo está separado del siguiente con una coma.

Cuando se ejecuta la expresión `match`, compara el valor resultante con el patrón de cada brazo, en orden. Si un patrón coincide con el valor, se ejecuta el código asociado con ese patrón. Si ese patrón no coincide con el valor, la ejecución continúa con el siguiente brazo, como en una máquina clasificadora de monedas. Podemos tener tantos brazos como necesitemos: en el ejemplo anterior, nuestro `match` tiene cuatro brazos.

En Cairo, el orden de los brazos debe seguir el mismo orden que el enum.

El código asociado con cada brazo es una expresión, y el valor resultante de la expresión en el brazo coincidente es el valor que se devuelve para toda la expresión `match`.

Normalmente no usamos llaves si el código del brazo del `match` es corto, como en nuestro ejemplo donde cada brazo simplemente devuelve un valor. Si desea ejecutar varias líneas de código en un brazo del `match`, debe usar llaves, con una coma después del brazo. Por ejemplo, el siguiente código imprime "¡Moneda de la suerte!" cada vez que se llama al método con una `Coin::Penny(())`, pero aún devuelve el último valor del bloque, `1`: 

```rust
fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => {
            ('Lucky penny!').print();
            1
        },
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(_)=> 25,
    }
}
```

## Patrones que se vinculan con valores

Otra característica útil de los brazos de coincidencia es que pueden vincularse con las partes de los valores que coinciden con el patrón. Así es como podemos extraer valores de las variantes de una enum.

Como ejemplo, cambiemos una de nuestras variantes de enum para que contenga datos en su interior. Desde 1999 hasta 2008, la Casa de la Moneda de los Estados Unidos acuñó monedas de 25 centavos con diseños diferentes para cada uno de los 50 estados en un lado. Ninguna otra moneda tenía diseños estatales, por lo que solo los cuartos tienen este valor adicional. Podemos agregar esta información a nuestra `enum` cambiando la variante `Quarter` para incluir un valor `UsState` almacenado en su interior, lo cual hemos hecho en la Lista 5-4.

```rust
#[derive(Drop)]
enum UsState {
    Alabama: (),
    Alaska: (),
}

#[derive(Drop)]
enum Coin {
    Penny: (),
    Nickel: (),
    Dime: (),
    Quarter: (UsState),
}
```

Listado 5-4: Un enum `Coin` en el que la variante `Quarter` también tiene un valor `UsState`

Imaginemos que un amigo está tratando de recolectar todas las 50 monedas de cuarto de estado. Mientras clasificamos nuestro cambio suelto por tipo de moneda, también llamaremos el nombre del estado asociado con cada cuarto para que si es uno que nuestro amigo no tiene, puedan agregarlo a su colección.

En la expresión `match` de este código, agregamos una variable llamada `state` al patrón que coincide con los valores de la variante `Coin::Quarter`. Cuando se hace una coincidencia de `Coin::Quarter`, la variable `state` se vinculará al valor del estado de ese cuarto. Luego podemos usar `state` en el código para ese brazo, así:

```rust
fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => 1,
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(state)=> {
            state.print();
            25
        },
    }
}
```

Para imprimir el valor de una variante de un enum en Cairo, necesitamos agregar una implementación para la función `print` de `debug::PrintTrait`:

```rust
impl UsStatePrintImpl of PrintTrait::<UsState> {
    fn print(self: UsState) {
        match self {
            UsState::Alabama(_) => ('Alabama').print(),
            UsState::Alaska(_) => ('Alaska').print(),
        }
    }
}
```

Si llamáramos a `value_in_cents(Coin::Quarter(UsState::Alaska(())))`, `coin` sería `Coin::Quarter(UsState::Alaska())`. Cuando comparamos ese valor con cada uno de los brazos del `match`, ninguno coincide hasta que llegamos a `Coin::Quarter (state)`. En ese momento, la asignación para `state` será el valor `UsState::Alaska()`. Luego podemos usar esa asignación en el `PrintTrait`, obteniendo así el valor interno de estado fuera de la variante `Coin` para `Quarter`.

## Coincidencia Con Opciones

En la sección anterior, queríamos obtener el valor interno `T` fuera del caso `Some` al usar `Option<T>`; ¡también podemos manejar `Option<T>` usando `match`, como lo hicimos con el `enum` `Coin`! En lugar de comparar monedas, compararemos las variantes de `Option<T>`, pero la forma en que funciona la expresión `match` sigue siendo la misma. Puedes usar opciones importando el trait `option::OptionTrait`.

Digamos que queremos escribir una función que tome una `Option<u8>` y, si hay un valor dentro, agregue `1_u8` a ese valor. Si no hay un valor dentro, la función debería devolver el valor `None` y no intentar realizar ninguna operación.

Esta función es muy fácil de escribir, gracias a `match`, y se verá como en el listado 5-5.

```rust
use option::OptionTrait;
use debug::PrintTrait;

fn plus_one(x: Option<u8>) -> Option<u8> {
    match x {
        Option::Some(val) => Option::Some(val + 1_u8),
        Option::None(_) => Option::None(()),
    }
}

fn main() {
    let five : Option<u8> = Option::Some(5_u8);
    let six : Option<u8> = plus_one(five);
    six.unwrap().print();
    let none = plus_one(Option::None(()));
    none.unwrap().print();
}
```

Listado 5-5: Una función que usa una expresión `match` en un `Option<u8>`

Tenga en cuenta que los brazos (`arms`) deben respetar el mismo orden que el enum definido en `OptionTrait` de la librería central de Cairo.

```rust
    enum Option<T> {
        Some: T,
        None: (),
    }
```

Estudiemos con más detalle la primera ejecución de `plus_one`. Cuando llamamos a `plus_one(five)`, la variable `x` en el cuerpo de `plus_one` tendrá el valor `Some(5_u8)`. Luego, lo comparamos con cada rama del `match`:

```rust
    Option::Some(val) => Option::Some(val + 1_u8),
```

¿El valor `Option::Some(5_u8)` coincide con el patrón `Option::Some(val)`? ¡Sí! Tenemos la misma variante. `val` se vincula al valor contenido en `Option::Some`, por lo que `val` toma el valor `5_u8`. Luego se ejecuta el código en el brazo del `match`, por lo que agregamos `1_u8` al valor de `val` y creamos un nuevo valor `Option::Some` con nuestro total `6_u8` en su interior. Debido a que se ha realizado la primera coincidencia, no se comparan otros brazos.

Ahora consideremos la segunda llamada de `plus_one` en nuestra función principal, donde `x` es `Option::None(())`. Entramos en el `match` y comparamos con el primer brazo:

```rust
    Option::Some(val) => Option::Some(val + 1_u8),
```

El valor `Option::Some(5_u8)` no coincide con el patrón `Option::None`, así que continuamos con el siguiente brazo:

```rust
    Option::None(_) => Option::None(()),
```

¡Coincide! No hay valor al que agregar, por lo que el programa se detiene y devuelve el valor `Option::None(())` en el lado derecho de `=>`.

Combinar `match` y enumeraciones es útil en muchas situaciones. Verás este patrón mucho en el código de Cairo: `match` contra una enumeración, enlaza una variable con los datos internos y luego ejecuta código basado en ella. Es un poco complicado al principio, pero una vez que te acostumbras, desearás tenerlo en todos los lenguajes. Es consistentemente favorito de los usuarios.

## Los Matches Son Exhaustivos

Hay otro aspecto de los matches que necesitamos discutir: los patrones de los brazos deben cubrir todas las posibilidades. Considera esta versión de nuestra función `plus_one`, que tiene un error y no se compilará:
```bash
$ cairo-run src/test.cairo
    error: Unsupported match. Currently, matches require one arm per variant,
    in the order of variant definition.
    --> test.cairo:34:5
        match x {
        ^*******^
    Error: failed to compile: ./src/test.cairo
```

Cairo sabe que no cubrimos todos los casos posibles, ¡e incluso sabe qué patrón olvidamos! Los matches en Cairo son exhaustivos: debemos cubrir todas las posibilidades para que el código sea válido. Especialmente en el caso de `Option<T>`, cuando Cairo nos impide olvidar manejar explícitamente el caso `None`, nos protege de asumir que tenemos un valor cuando podríamos tener nulo, lo que hace imposible el error de mil millones de dólares discutido anteriormente.

## Match 0 y el Comodín \_

Usando enums, también podemos tomar acciones especiales para algunos valores particulares, pero para todos los demás valores tomar una acción predeterminada. Actualmente solo se admiten `0` y el operador `_`.

Imaginemos que estamos implementando un juego en el que obtienes un número aleatorio entre 0 y 7. Si tienes 0, ganas. Para todos los demás valores pierdes. Aquí hay un match que implementa esa lógica, con el número codificado en lugar de un valor aleatorio.

```rust
fn did_i_win(nb: felt252) {
    match nb {
        0 => ('You won!').print(),
        _ => ('You lost...').print(),
    }
}
```

La primera armadura tiene un patrón de valores literales 0. Para la última armadura que cubre todos los demás posibles valores, el patrón es el carácter `_`. Este código se compila, aunque no hayamos listado todos los posibles valores que un `felt252` puede tener, ya que el último patrón coincidirá con todos los valores que no estén específicamente enumerados. Este patrón de captura de todo cumple con el requisito de que `match` debe ser exhaustivo. Tenga en cuenta que debemos poner la armadura de captura de todo al final porque los patrones se evalúan en orden. Si ponemos la armadura de captura de todo antes, las otras armaduras nunca se ejecutarán, por lo que Cairo nos advertirá si agregamos armaduras después de una de captura de todo.

<!-- TODO: puede que necesitemos enlazar el final de este capítulo con el capítulo de patrones y coincidencias -->
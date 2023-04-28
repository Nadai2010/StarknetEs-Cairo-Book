## ¿Qué es Ownership?

Cairo implementa un sistema de propiedad para garantizar la seguridad y corrección de su código compilado. El mecanismo de propiedad complementa el sistema de tipos lineales, que obliga a que los objetos se usen exactamente una vez. Esto ayuda a prevenir operaciones comunes que pueden producir errores en tiempo de ejecución, como referencias ilegales de direcciones de memoria o múltiples escrituras en la misma dirección de memoria, y garantiza la corrección de los programas de Cairo comprobando en tiempo de compilación que todos los diccionarios están aplastados.

Ahora que hemos pasado la sintaxis básica de Cairo, no incluiremos todo el código `fn main() {` en los ejemplos, así que si estás siguiendo, asegúrate de colocar los siguientes ejemplos dentro de una función `main` manualmente. Como resultado, nuestros ejemplos serán un poco más concisos, lo que nos permitirá enfocarnos en los detalles reales en lugar del código de plantilla.

### Reglas de Ownership

En primer lugar, echemos un vistazo a las reglas de propiedad. Mantenga estas reglas en mente mientras trabajamos a través de los ejemplos que las ilustran:

- Cada valor en Cairo tiene un _propietario_.
- Solo puede haber un propietario a la vez.
- Cuando el propietario sale del ámbito, el valor será _descartado_.

### Ámbito de Variables

Como primer ejemplo de propiedad, veremos el _ámbito_ de algunas variables. Un ámbito es el alcance dentro de un programa para el cual un elemento es válido. Tomemos la siguiente variable:

```rust
let s = 'hello';
```

La variable `s` hace referencia a una cadena corta, donde el valor de la cadena está codificado en el texto de nuestro programa. La variable es válida desde el momento en que se declara hasta el final del _ámbito_ actual. La Lista 3-1 muestra un programa con comentarios que anotan dónde sería válida la variable `s`.

```rust
    {                      // s is not valid here, it’s not yet declared
        let s = 'hello';   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

<span class="caption">Lista 3-1: Una variable y el ámbito en el que es válida</span>

En otras palabras, hay dos puntos importantes en el tiempo aquí:

- Cuando `s` entra en el _ámbito_, es válida.
- Permanece válida hasta que sale del _ámbito_.

En este punto, la relación entre los ámbitos y cuándo las variables son válidas es similar a la de otros lenguajes de programación. Ahora construiremos sobre esta comprensión introduciendo el tipo de datos `Array`.

### El tipo `Array`

Para ilustrar las reglas de propiedad, necesitamos un tipo de datos que sea más complejo que los que cubrimos en la sección de [Tipos de Datos][data-types]<!-- ignore --> del Capítulo 3. Los tipos cubiertos anteriormente tienen un tamaño conocido, se pueden copiar rápida y trivialmente para crear una nueva instancia independiente si otra parte del código necesita usar el mismo valor en un ámbito diferente, y se pueden descartar fácilmente cuando ya no se usan. Pero queremos examinar datos cuyo tamaño es desconocido en tiempo de compilación y que no se pueden copiar trivialmente: el tipo `Array`.

En Cairo, cada celda de memoria solo se puede escribir una vez. Los arrays se representan en memoria mediante un segmento de celdas de memoria contiguas, y el sistema de tipos lineales de Cairo se utiliza para garantizar que cada celda nunca se escriba más de una vez.
Considere el siguiente código, en el que definimos una variable `arr` de tipo `Array` que contiene valores `u128`:

```rust
use array::ArrayTrait;
...

let arr = ArrayTrait::<u128>::new();
```

Puede agregar valores a un `Array` utilizando el método `append`:

```rust
let mut arr = ArrayTrait::<u128>::new();
arr.append(1);
arr.append(2);
```

Entonces, ¿cómo garantiza el sistema de propiedad que cada celda nunca se escriba más de una vez?
Considere el siguiente código, en el que intentamos pasar la misma instancia de un array en dos llamadas de función consecutivas:

```rust
use array::ArrayTrait;
fn foo(arr: Array<u128>) {
}

fn bar(arr:Array<u128>){
}

fn main() {
    let mut arr = ArrayTrait::<u128>::new();
    foo(arr);
    bar(arr);
}
```

En este caso, intentamos pasar la misma instancia de matriz `arr` por valor a las funciones `foo` y `bar`, lo que significa que el parámetro utilizado en ambas llamadas de función es la misma instancia de la matriz. Si agrega un valor a la matriz en `foo` y luego intenta agregar otro valor a la misma matriz en `bar`, lo que sucederá es que intentará escribir en la misma celda de memoria dos veces, lo que no está permitido en Cairo.
Para evitar esto, la propiedad de la variable `arr` se mueve de la función `main` a la función `foo`. Cuando se intenta llamar a `bar` con `arr` como parámetro, la propiedad de `arr` ya se movió a la primera llamada. El sistema de propiedad nos impide usar la misma instancia de `arr` en `foo`.

Ejecutar el código anterior resultará en un error en tiempo de compilación:

```console
error: Variable was previously moved. Trait has no implementation in context: core::traits::Copy::<core::array::Array::<core::integer::u128>>
 --> array.cairo:6:9
    let mut arr = ArrayTrait::<u128>::new();
        ^*****^
```

### El Trait `Copy`

Si un tipo implementa el trait `Copy`, pasar su valor a una función no moverá la propiedad del valor a la función llamada, sino que pasará una copia del valor.
Puedes implementar el trait `Copy` en tu tipo agregando la anotación `#[derive(Copy)]` a la definición de tu tipo. Sin embargo, Cairo no permitirá que un tipo sea anotado con `Copy` si el tipo en sí mismo o cualquiera de sus componentes no implementan el trait `Copy`.
Mientras que los Arrays y Diccionarios no pueden ser copiados, los tipos personalizados que no los contienen sí pueden serlo.

```rust
#[derive(Copy, Drop)]
struct Point {
    x: u128,
    y: u128,
}

fn main() {
    let p1 = Point { x: 5, y: 10 };
    foo(p1);
    foo(p1);
}

fn foo(p: Point) {
    // do something with p
}
```

En este ejemplo, podemos pasar `p1` dos veces a la función `foo` porque el tipo `Point` implementa el trait `Copy`. Esto significa que cuando pasamos `p1` a `foo`, en realidad estamos pasando una copia de `p1`, y la propiedad de `p1` permanece en la función principal.

Si eliminamos la derivación del trait `Copy` del tipo `Point`, obtendremos un error en tiempo de compilación al intentar compilar el código.

### El Trait `Drop`

Es posible que hayas notado que el tipo `Point` en el ejemplo anterior también implementa el trait `Drop`. En Cairo, un valor no puede salir del ámbito a menos que se haya movido previamente.
Por ejemplo, el siguiente código no se compilará porque la estructura `A` no se mueve antes de que salga del ámbito:

```rust
struct A {}

fn main() {
    A {}; // error: Value not dropped.
}
```

En Cairo, esto se hace para garantizar la solidez de los programas. La solidez se refiere al hecho de que si una declaración durante la ejecución del programa es falsa, ningún probador deshonesto puede convencer a un verificador honesto de que es verdadera. En nuestro caso, queremos asegurar la consistencia de las actualizaciones consecutivas de claves de un diccionario durante la ejecución del programa, lo cual solo se verifica cuando los diccionarios se "aplastan" - lo que mueve la propiedad del diccionario al método `squash`, permitiendo que el diccionario salga de ámbito. Los diccionarios no "aplastados" son peligrosos, ya que un probador malintencionado podría probar la corrección de actualizaciones inconsistentes.

Sin embargo, los tipos que implementan el trait `Drop` se permiten que salgan de ámbito sin ser movidos explícitamente. Cuando un valor de un tipo que implementa el trait `Drop` sale de ámbito, se llama a la implementación `Drop` en el tipo, lo que mueve el valor a la función `drop`, permitiendo que salga de ámbito: esto es lo que llamamos "eliminar" un valor. Es importante tener en cuenta que la implementación de `Drop` es una "operación nula", lo que significa que no realiza ninguna acción aparte de permitir que el valor salga de ámbito.

La implementación de `Drop` se puede derivar para todos los tipos, lo que les permite eliminarse al salir de ámbito, excepto para los diccionarios (`Felt252Dict`) y los tipos que contienen diccionarios.
Por ejemplo, el siguiente código compila:

```rust
#[derive(Drop)]
struct A {}

fn main() {
    A {}; // Now there is no error.
}
```

### El trait `Destruct`

Llamar manualmente al método `squash` en un diccionario no es muy conveniente y es fácil de olvidar hacerlo. Para facilitar el uso de los diccionarios, Cairo proporciona el trait `Destruct`, que te permite especificar el comportamiento de un tipo cuando sale del ámbito. Si bien los diccionarios no implementan el trait `Drop`, sí implementan el trait `Destruct`, lo que les permite ser `aplastados` automáticamente cuando salen del ámbito. Esto significa que puedes usar diccionarios sin tener que llamar manualmente al método `squash`.

Considera el siguiente ejemplo, en el que definimos un tipo personalizado que contiene un diccionario:

```rust
use dict::Felt252DictTrait;

struct A {
    dict: Felt252Dict<u128>
}

fn main() {
    A {
        dict: Felt252DictTrait::new()
    };
}
```

Si intenta ejecutar este código, obtendrá un error de tiempo de compilación:

```console
error: Variable not dropped. Trait has no implementation in context: core::traits::Drop::<temp7::temp7::A>. Trait has no implementation in context: core::traits::Destruct::<temp7::temp7::A>.
 --> temp7.cairo:7:5
    A {
    ^*^
```

Cuando `A` sale del alcance, no puede ser liberado ya que no implementa ni el `Drop` (ya que contiene un diccionario y no puede `derive(Drop)`) ni el trait `Destruct`. Para solucionar esto, podemos derivar la implementación del trait `Destruct` para el tipo `A`:

```rust
use dict::Felt252DictTrait;

#[derive(Destruct)]
struct A {
    dict: Felt252Dict<u128>
}

fn main() {
    A {
        dict: Felt252DictTrait::new()
    }; // No error here
}
```

### Copiar datos de un Array con Clone

Si queremos copiar profundamente los datos de un `Array`, podemos utilizar un método común llamado `clone`. Discutiremos la sintaxis de los métodos en el Capítulo 5, pero como los métodos son una característica común en muchos lenguajes de programación, es probable que ya los hayas visto antes.

Aquí hay un ejemplo del método `clone` en acción.

> Nota: en el siguiente ejemplo, necesitamos importar el rasgo `Clone` del módulo `clone` de la biblioteca estándar, y su implementación para el tipo `array` del módulo `array`.

```rust
use array::ArrayTrait;
use clone::Clone;
use array::ArrayTCloneImpl;
...
let arr1 = ArrayTrait::new::<u128>();
let arr2 = arr1.clone();

```

> Nota: necesitarás ejecutar `cairo-run` con la opción `--available-gas=2000000` para ejecutar este ejemplo, ya que utiliza un bucle y debe ser ejecutado con un límite de gas.

Cuando ves una llamada a `clone`, sabes que se está ejecutando algún código arbitrario y ese código puede ser costoso. Es un indicador visual de que algo diferente está sucediendo.

### Propiedad y Funciones

Pasar una variable a una función puede moverla o copiarla. Como se vio en la sección de Array, pasar un `Array` como parámetro de función transfiere su propiedad; veamos qué sucede con otros tipos.

El Listado 3-3 tiene un ejemplo con algunas anotaciones que muestran dónde las variables entran y salen de ámbito.

<span class="filename">Nombre de archivo: src/main.cairo</span>

```rust
#[derive(Drop)]
struct MyStruct{}

fn main() {
    let my_struct = MyStruct{};  // my_struct comes into scope

    takes_ownership(my_struct);             // my_struct's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5_u128;                 // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but u128 implements Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope and is dropped. But because my_struct's value was moved, nothing
// special happens.

fn takes_ownership(some_struct: A) { // some_struct comes into scope
} // Here, some_struct goes out of scope and `drop` is called.

fn makes_copy(some_uinteger: u128) { // some_uinteger comes into scope
} // Here, some_integer goes out of scope and is dropped.
```

<span class="caption">Listado 3-3: Funciones con propiedad y alcance anotados</span>

Si intentamos usar `my_struct` después de la llamada a `takes_ownership`, Cairo lanzará un error en tiempo de compilación. Estas verificaciones estáticas nos protegen de errores. Intenta agregar código a `main` que use `my_struct` y `x` para ver dónde puedes usarlos y dónde las reglas de propiedad te impiden hacerlo.

### Valores de retorno y alcance

La devolución de valores también puede transferir la propiedad. El Listado 3-4 muestra un ejemplo de una función que devuelve algún valor, con anotaciones similares a las del Listado 3-3.

<span class="filename">Nombre de archivo: src/main.cairo</span>

```rust
#[derive(Drop)]
struct A{}

fn main() {
    let a1 = gives_ownership();         // gives_ownership moves its return
                                        // value into a1

    let a2 = A{};     // a2 comes into scope

    let a3 = takes_and_gives_back(a2);  // a2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into a3
} // Here, a3 goes out of scope and is dropped. a2 was moved, so nothing
  // happens. a1 goes out of scope and is dropped.

fn gives_ownership() -> A {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_a = A{}; // some_a comes into scope

    some_a                              // some_a is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes an instance of A and returns one
fn takes_and_gives_back(some_a: A) -> A { // some_a comes into
                                                      // scope

    some_a  // some_a is returned and moves out to the calling function
}
```

<span class="caption">Listado 3-4: Transferencia de propiedad de valores devueltos</span>

Cuando una variable sale del ámbito, su valor se elimina, a menos que la propiedad del valor se haya transferido a otra variable.

Si bien esto funciona, tomar propiedad y luego devolver la propiedad con cada función es un poco tedioso. ¿Qué sucede si queremos permitir que una función use un valor pero no tome posesión de él? Es bastante molesto que todo lo que pasemos también deba ser devuelto si queremos usarlo nuevamente, además de cualquier dato que resulte del cuerpo de la función que también podríamos querer devolver.

Cairo nos permite devolver varios valores usando una tupla, como se muestra en el Listado 3-5. 

<span class="filename">Nombre de archivo: src/main.cairo</span>

```rust
use array::ArrayTrait;
fn main() {
    let arr1 = ArrayTrait::<u128>::new();

    let (arr2, len) = calculate_length(arr1);
}

fn calculate_length(arr: Array<u128>) -> (Array<u128>, usize) {
    let length = arr.len(); // len() returns the length of an array

    (arr, length)
}
```

<span class="caption">Listado 3-5: Devolviendo propiedad de los parámetros</span>

Pero esto es demasiado ceremonioso y mucho trabajo para un concepto que debería ser común. Afortunadamente, Cairo tiene dos características para usar un valor sin transferir la propiedad, llamadas _referencias_ y _snapshots_.

[data-types]: ch02-02-data-types.html#data-types
[method-syntax]: ch04-03-method-syntax.html#method-syntax

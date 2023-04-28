## Referencias y Snapshots

El problema con el código de tupla en el Listado 3-5 es que tenemos que devolver el `Array` a la función de llamada para que podamos seguir usando el `Array` después de la llamada a `calculate_length`, ya que el `Array` se movió a `calculate_length`.

### Snapshots

En su lugar, podemos proporcionar una _instantánea_ del valor `Array`. En Cairo, una instantánea es una vista inmutable de un valor en un cierto momento en el tiempo. En el capítulo anterior, hablamos de cómo el sistema de propiedad de Cairo nos impide usar un valor después de haberlo movido, protegiéndonos de escribir potencialmente dos veces en la misma celda de memoria al agregar valores a los arreglos. Sin embargo, no es muy conveniente. Veamos cómo podemos mantener la propiedad del valor en la función de llamada usando instantáneas.

Aquí es cómo definiría y usaría una función `calculate_length` que toma una instantánea de un arreglo como parámetro en lugar de tomar propiedad del valor subyacente. En este ejemplo, la función `calculate_length` devuelve la longitud del arreglo pasado como parámetro. Como lo estamos pasando como una instantánea, que es una vista inmutable del arreglo, podemos estar seguros de que la función `calculate_length` no mutará el arreglo, y la propiedad del arreglo se mantiene en la función principal.

<span class="filename">Nombre de archivo: src/lib.cairo</span>

```rust
use array::ArrayTrait;
use debug::PrintTrait;

fn main() {
    let mut arr1 = ArrayTrait::<u128>::new();
    let first_snapshot = @arr1; // Take a snapshot of `arr1` at this point in time
    arr1.append(1_u128); // Mutate `arr1` by appending a value
    let first_length = calculate_length(first_snapshot); // Calculate the length of the array when the snapshot was taken
    let second_length = calculate_length(@arr1); // Calculate the current length of the array
    first_length.print();
    second_length.print();
}

fn calculate_length(arr: @Array<u128>) -> usize {
    arr.len()
}
```

> Nota: Solo es posible llamar al método `len()` en un snapshot de un array porque está definido así en el trait `ArrayTrait`. Si intentas llamar a un método que no está definido para snapshots en un snapshot, obtendrás un error de compilación. Sin embargo, puedes llamar a métodos que esperan un snapshot en tipos que no son snapshots. 

La salida de este programa es:

```console
[DEBUG]	                               	(raw: 0)

[DEBUG]	                              	(raw: 1)

Run completed successfully, returning []
```

La primera observación es que todo el código de tuplas en la declaración de variables y en el valor de retorno de la función ha desaparecido. La segunda observación es que pasamos `@arr1` a `calculate_length` y, en su definición, tomamos `@Array<u128>` en lugar de `Array<u128>`.

Veamos más de cerca la llamada a la función aquí:

```rust
let mut arr1 = ArrayTrait::<u128>::new();
let second_length = calculate_length(@arr1); // Calculate the current length of the array
```

La sintaxis `@arr1` nos permite crear una instantánea (snapshot) del valor en `arr1`. Como una instantánea es una vista inmutable de un valor, el valor al que apunta no puede ser modificado a través de la instantánea y el valor al que se refiere no será eliminado una vez que la instantánea deje de ser usada.

De manera similar, la firma de la función utiliza `@` para indicar que el tipo del parámetro `arr` es una instantánea. Añadamos algunas anotaciones explicativas:

```rust
fn calculate_length(array_snapshot: @Array<u128>) -> usize { // array_snapshot is a snapshot of an Array
    array_snapshot.len()
} // Here, array_snapshot goes out of scope and is dropped.
// However, because it is only a view of what the original array `arr` contains, the original `arr` can still be used.
```

El alcance en el que la variable `array_snapshot` es válida es el mismo que el alcance de cualquier parámetro de función, pero el valor subyacente del snapshot no se eliminará cuando `array_snapshot` deje de usarse. Cuando las funciones tienen snapshots como parámetros en lugar de los valores reales, no necesitamos devolver los valores para devolver la propiedad del valor original, porque nunca la tuvimos.

Los snapshots se pueden convertir de nuevo en valores regulares usando el operador `desnap` `*`, siempre y cuando el tipo de valor sea copiable (lo cual no es el caso para los Arrays, ya que no implementan `Copy`). En el siguiente ejemplo, queremos calcular el área de un rectángulo, pero no queremos tomar la propiedad del rectángulo en la función `calculate_area`, porque podríamos querer usar el rectángulo de nuevo después de la llamada a la función. Dado que nuestra función no muta la instancia del rectángulo, podemos pasar el snapshot del rectángulo a la función, y luego transformar los snapshots de nuevo en valores usando el operador `desnap` `*`.

El tipo de snapshot siempre es copiable y eliminable, para que pueda usarlo varias veces sin preocuparse por las transferencias de propiedad.

```rust
#[derive(Copy,Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

fn main(){
    let rec = Rectangle{height:3_u64, width:10_u64};
}

fn calculate_area(rec: @Rectangle) -> u64 {
    // As rec is a snapshot to a Rectangle, its fields are also snapshots of the fields types.
    // We need to transform the snapshots back into values using the desnap operator `*`.
    // This is only possible if the type is copyable, which is the case for u64.
    // Here, `*` is used for both multiplying the height and width and for desnapping the snapshots.
    *rec.height * *rec.width
}
```

Pero, ¿qué sucede si intentamos modificar algo que estamos pasando como instantánea? Prueba el código en la Lista 3-6. ¡Alerta de spoiler: no funciona!

<span class="filename">Nombre de archivo: src/lib.cairo</span>

```rust
#[derive(Copy,Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

fn main(){
    let rec = Rectangle{height:3_u64, width:10_u64};
    flip(@rec);
}

fn flip(rec: @Rectangle) {
    let temp = rec.height;
    rec.height = rec.width;
    rec.width = temp;
}
```

<span class="caption">Listado 3-6: Intentando modificar un valor de snapshot</span>

Aquí está el error:

```console
error: Invalid left-hand side of assignment.
 --> ownership.cairo:15:5
    rec.height = rec.width;
    ^********^
```

### Referencias mutables

Podemos lograr el comportamiento que queremos en el Listado 3-6 utilizando una _referencia mutable_ en lugar de un snapshot. Las referencias mutables son valores mutables pasados a una función que se devuelven implícitamente al final de la función, devolviendo la propiedad al contexto de llamada. Al hacerlo, le permiten mutar el valor pasado y mantener su propiedad devolviéndolo automáticamente al final de la ejecución.

En Cairo, se puede pasar un parámetro como _referencia mutable_ utilizando el modificador `ref`.

> **Nota**: En Cairo, un parámetro solo se puede pasar como _referencia mutable_ utilizando el modificador `ref` si la variable se declara como mutable con `mut`.

En el Listado 3-7, usamos una referencia mutable para modificar el valor del campo `height` de la instancia de `Rectangle` en la función `flip`.

```rust
use debug::PrintTrait;
#[derive(Copy,Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

fn main(){
    let mut rec = Rectangle{height:3_u64, width:10_u64};
    flip(ref rec);
    rec.height.print();
    rec.width.print();
}

fn flip(ref rec: Rectangle) {
    let temp = rec.height;
    rec.height = rec.width;
    rec.width = temp;
}
```

Primero, cambiamos `rec` a `mut`. Luego, pasamos una referencia mutable de `rec` a `flip` con `ref rec` y actualizamos la firma de la función para aceptar una referencia mutable con `ref rec: Rectangle`. Esto deja muy claro que la función `flip` modificará el valor de la instancia de `Rectangle` pasada como parámetro.

La salida del programa es:

```console
[DEBUG]
                                (raw: 10)

[DEBUG]	                        (raw: 3)
```

Como resumen, lo que hemos discutido acerca de ownership, snapshots y las referencias es:

- En un momento dado, una variable solo puede tener un propietario.
- Puedes pasar una variable por valor, por instantánea o por referencia a una función.
- Si pasas una variable por valor, la propiedad de la variable se transfiere a la función.
- Si quieres mantener la propiedad de la variable y sabes que tu función no la va a modificar, puedes pasarla como instantánea con `@`.
- Si quieres mantener la propiedad de la variable y sabes que tu función la modificará, puedes pasarla como referencia mutable con `ref`.
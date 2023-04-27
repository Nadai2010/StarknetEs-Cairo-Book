# Enums y Coincidencia de Patrones

Los enums, abreviatura de "enumeraciones", son una forma de definir un tipo de datos personalizado que consiste en un conjunto fijo de valores nombrados, llamados _variantes_. Los enums son útiles para representar una colección de valores relacionados donde cada valor es distinto y tiene un significado específico.

## Variantes y Valores de Enum

Aquí hay un ejemplo sencillo de un enum:

```rs
#[derive(Drop)]
enum Direction {
    North : (),
    East : (),
    South : (),
    West : (),
}
```

A diferencia de otros lenguajes como Rust, cada variante tiene un tipo. En este ejemplo, hemos definido un enum llamado `Direction` con cuatro variantes: `North`, `East`, `South` y `West`. La convención de nomenclatura es utilizar PascalCase para las variantes del enum. Cada variante representa un valor distinto del tipo `Direction` y está asociada con un tipo unitario `()`. Una variante puede ser instanciada utilizando esta sintaxis:

```rs
let direction = Direction::North(());
```

Es fácil escribir código que se comporte de manera diferente según la variante de una instancia de un enum, como en este ejemplo, donde se ejecuta un código específico según una dirección. Puedes obtener más información sobre esto en la página [The Match Control Flow Construct](ch05-02-the-match-control-flow-construct.md).
## Enums combinados con tipos personalizados

Los enums también pueden ser utilizados para almacenar datos más interesantes asociados con cada variante. Por ejemplo:

```rs
#[derive(Drop)]
enum Message {
    Quit : (),
    Echo : (felt252),
    Move : (u128, u128),
}
```

En este ejemplo, el enum `Message` tiene tres variantes: `Quit`, `Echo` y `Move`, todas con tipos diferentes:

- `Quit` no tiene datos asociados en absoluto.
- `Echo` incluye un solo campo.
- `Move` incluye dos valores u128.

Incluso puedes usar una estructura o otro enum que hayas definido dentro de una de las variantes de tu enum.

## Implementaciones de Traits para Enums

En Cairo, puedes definir traits e implementarlos para tus enums personalizados. Esto te permite definir métodos y comportamientos asociados con el enum. Aquí hay un ejemplo de cómo definir un trait e implementarlo para el enum `Message` anterior:

```rs
trait Processing {
    fn process(self: Message);
}

impl ProcessingImpl of Processing {
    fn process(self: Message) {
        match self {
            Message::Quit(()) => {
                'quitting'.print();
            },
            Message::Echo(value) => {
                value.print();
            },
            Message::Move((x, y)) => {
                'moving'.print();
            },
        }
    }
}
```

En este ejemplo, implementamos el trait `Processing` para `Message`. Así es cómo podría ser utilizado para procesar un mensaje Quit:

```rust
let msg: Message = Message::Quit(());
msg.process();
```

Al ejecutar este código se imprimiría `quitting`.

## El Enum Option y sus ventajas

El enum Option es un enum estándar en Cairo que representa el concepto de un valor opcional. Tiene dos variantes: `Some: T` y `None: ()`. `Some: T` indica que hay un valor de tipo `T`, mientras que `None` representa la ausencia de un valor.

```rs
enum Option<T> {
    Some: T,
    None: (),
}
```

El enum `Option` es útil porque te permite representar explícitamente la posibilidad de que un valor esté ausente, lo que hace que tu código sea más expresivo y fácil de entender. Usar `Option` también puede ayudar a prevenir errores causados por el uso de valores `null` no inicializados o inesperados.

Para darte un ejemplo, aquí hay una función que devuelve el índice del primer elemento de un arreglo con un valor dado, o `None` si el elemento no está presente.

> Nota: en el futuro sería bueno reemplazar este ejemplo con algo más simple que use un ciclo y sin código relacionado con el gas.

```rust
use array::ArrayTrait;
use debug::PrintTrait;
fn find_value_recursive(
    arr: @Array<felt252>, value: felt252, index: usize
) -> Option<usize> {

    match gas::withdraw_gas() {
        Option::Some(_) => {},
        Option::None(_) => {
            let mut data = ArrayTrait::new();
            data.append('OOG');
            panic(data);
        },
    }

    if index >= arr.len() {
        return Option::None(());
    }

    if *arr.at(index) == value {
        return Option::Some(index);
    }

    find_value_recursive(arr, value, index + 1_usize)
}

#[test]
#[available_gas(999999)]
fn test_increase_amount() {
    let mut my_array = ArrayTrait::new();
    my_array.append(3);
    my_array.append(7);
    my_array.append(2);
    my_array.append(5);

    let value_to_find = 7;
    let result = find_value_recursive(@my_array, value_to_find, 0_usize);

    match result {
        Option::Some(index) => {
            if index == 1_usize {
                'it worked'.print();
            }
        },
        Option::None(()) => {
            'not found'.print();
        },
    }
}
```

Al ejecutar este código se imprimiría `it worked`.

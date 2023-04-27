# Tipos de datos genéricos

Usamos genéricos para crear definiciones de declaraciones de elementos, como structs y funciones, que luego podemos usar con muchos tipos de datos concretos diferentes. ¡En Cairo podemos usar genéricos al definir funciones, structs, enums, traits, implementaciones y métodos! En este capítulo vamos a ver cómo usar efectivamente tipos genéricos con todos ellos.

## Funciones genéricas

Al definir una función que utiliza genéricos, colocamos los genéricos en la firma de la función, donde normalmente especificaríamos los tipos de datos del parámetro y el valor de retorno. Por ejemplo, imaginemos que queremos crear una función que, dadas dos matrices (`Array`) de elementos, devolverá la más grande. Si necesitamos realizar esta operación para listas de diferentes tipos, tendríamos que redefinir la función cada vez. Afortunadamente, podemos implementar la función una vez usando genéricos y seguir adelante con otras tareas.

```rust
// This code does not compile!

use array::ArrayTrait;

// Specify generic type T between the angulars
fn largest_list<T>(l1: Array<T>, l2: Array<T>) -> Array<T> {
    if l1.len() > l2.len() {
        l1
    } else {
        l2
    }
}

fn main() {
    let mut l1 = ArrayTrait::new();
    let mut l2 = ArrayTrait::new();

    l1.append(1);
    l1.append(2);

    l2.append(3);
    l2.append(4);
    l2.append(5);

    // There is no need to specify the concrete type of T because
    // it is inferred by the compiler
    let l3 = largest_list(l1, l2);
}
```

La función `largest_list` compara dos listas del mismo tipo y devuelve aquella con más elementos y elimina la otra. Si se compila el código anterior, se notará que fallará con un error diciendo que no se han definido traits para eliminar un array de un tipo genérico. Esto sucede porque el compilador no tiene forma de garantizar que un `Array<T>` sea eliminable al ejecutar la función `main`. Para eliminar un array de `T`, el compilador primero debe saber cómo eliminar `T`. Esto se puede solucionar especificando en la firma de la función `largest_list` que `T` debe implementar el trait de eliminación. La definición correcta de la función `largest_list` es la siguiente:

```rust
fn largest_list<T, impl TDrop: Drop<T>>(l1: Array<T>, l2: Array<T>) -> Array<T> {
    if l1.len() > l2.len() {
        l1
    } else {
        l2
    }
}
```

La nueva función `largest_list` incluye en su definición el requisito de que cualquier tipo genérico que se coloque allí debe poder eliminarse. La función `main` sigue sin cambios, el compilador es lo suficientemente inteligente como para deducir qué tipo concreto se está utilizando y si implementa el rasgo `Drop`.

### Restricciones para tipos genéricos

Al definir tipos genéricos, es útil tener información sobre ellos. Saber qué rasgos implementa un tipo genérico nos permite usarlos de manera más efectiva en la lógica de una función a costa de limitar los tipos genéricos que se pueden usar con la función. Vimos un ejemplo de esto anteriormente al agregar la implementación de `TDrop` como parte de los argumentos genéricos de `largest_list`. Si bien `TDrop` se agregó para cumplir con los requisitos del compilador, también podemos agregar restricciones para beneficiar nuestra lógica de función.

Imaginemos que queremos, dado una lista de elementos de algún tipo genérico `T`, encontrar el elemento más pequeño entre ellos. Inicialmente, sabemos que para que un elemento de tipo `T` sea comparable, debe implementar el rasgo `PartialOrd`. La función resultante sería:

```rust
// This code does not compile!
use array:ArrayTrait;

// Given a list of T get the smallest one.
// The PartialOrd trait implements comparison operations for T
fn smallest_element<T, impl TPartialOrd: PartialOrd<T>>(list: @Array<T>) -> T {
    // This represents the smallest element through the iteration
    // Notice that we use the desnap (*) operator
    let mut smallest = *list[0_usize];

    // The index we will use to move through the list
    let mut index = 1_usize;

    // Iterate through the whole list storing the smallest
    loop {
        if index >= list.len(){
            break smallest;
        }
        if *list[index] < smallest {
            smallest = *list[index];
        }
        index = index + 1;
    }
}

fn main()  {
    let mut list = ArrayTrait::new();
    list.append(5_u8);
    list.append(3_u8);
    list.append(10_u8);

    // We need to specify that we are passing a snapshot of `list` as an argument
    let s = smallest_element(@list);
    assert(s == 3_u8, 0);

}
```

La función `smallest_element` utiliza un tipo genérico `T` que implementa el rasgo `PartialOrd`, toma una instantánea de un `Array<T>` como parámetro y devuelve una copia del elemento más pequeño. Debido a que el parámetro es de tipo `@Array<T>`, ya no necesitamos soltarlo al final de la ejecución y por lo tanto no necesitamos implementar el rasgo `Drop` para `T` también. ¿Por qué entonces no compila?

Cuando hacemos indexación en `list`, el valor resultante es una instantánea del elemento indexado, a menos que `PartialOrd` esté implementado para `@T` necesitamos deshacer la instantánea del elemento usando `*`. La operación `*` requiere una copia de `@T` a `T`, lo que significa que `T` necesita implementar el rasgo `Copy`. Después de copiar un elemento de tipo `@T` a `T`, ahora hay variables con tipo `T` que necesitan ser soltadas, lo que requiere que `T` implemente también el rasgo `Drop`. Debemos entonces agregar la implementación de los rasgos `Drop` y `Copy` para que la función sea correcta. Después de actualizar la función `smallest_element`, el código resultante sería:

```rs
fn smallest_element<T, impl TPartialOrd: PartialOrd<T>, impl TCopy: Copy<T>, impl TDrop: Drop<T>>(list: @Array<T>) -> T {
    let mut smallest = *list[0_usize];
    let mut index = 1_usize;
    loop {
        if index >= list.len(){
            break smallest;
        }
        if *list[index] < smallest {
            smallest = *list[index];
        }
        index = index + 1;
    }
}
```

## Estructuras

También podemos definir estructuras que usen un parámetro de tipo genérico para uno o más campos usando la sintaxis `<>`, similar a las definiciones de funciones. Primero declaramos el nombre del parámetro de tipo dentro de los corchetes angulares justo después del nombre de la estructura. Luego usamos el tipo genérico en la definición de la estructura donde de otra manera especificaríamos tipos de datos concretos. El siguiente ejemplo de código muestra la definición de `Wallet<T>` que tiene un campo `balance` de tipo `T`.

```rust
// This code does not compile!

#[derive(Drop)]
struct Wallet<T> {
    balance: T,
}


fn main() {
   let w = Wallet{ balance: 3_u128};
}
```

La compilación del código anterior daría un error debido a que la macro `derive` no funciona bien con tipos genéricos. Cuando se usan tipos genéricos, es mejor escribir directamente los traits que se quieren utilizar:

```rust
struct Wallet<T> {
    balance: T,
}

impl WalletDrop<T, impl TDrop : Drop<T>> of Drop<Wallet<t>>;

fn main() {
   let w = Wallet{ balance: 3_u128};
}
```

Evitamos el uso de la macro `derive` para la implementación de `Drop` de `Wallet` y en su lugar definimos nuestra propia implementación de `WalletDrop`. Nótese que debemos definir, al igual que en las funciones, un tipo genérico adicional para `WalletDrop` diciendo que `T` también implementa el trait `Drop`. Básicamente estamos diciendo que la estructura `Wallet<T>` es dropeable siempre y cuando `T` también lo sea.

Finalmente, si queremos agregar un campo a `Wallet` que represente su dirección de Cairo y queremos que ese campo sea diferente a `T` pero también genérico, simplemente podemos agregar otro tipo genérico entre los `<>`:

```rust
struct Wallet<T, U> {
    balance: T,
    address: U,
}

impl WalletDrop<T, impl TDrop: Drop<T>, U, impl UDrop: Drop<U>> of Drop<Wallet<T, U>>;


fn main() {
   let w = Wallet{ balance: 3_u128, address: 14};
}
```

Agregamos a la definición de la estructura `Wallet` un nuevo tipo genérico `U` y luego asignamos este tipo al nuevo miembro del campo `address`. Luego adaptamos el trait `WalletDrop` para que funcione con el nuevo tipo genérico `U`. ¡Observa que al inicializar la estructura dentro de `main`, automáticamente infiere que `T` es un `u128` y `U` es un `felt252` y como ambos son droppable, `Wallet` también lo es!

## Enumeraciones

Como hicimos con las estructuras, podemos definir enumeraciones para contener tipos de datos genéricos en sus variantes. Por ejemplo, la enumeración `Option<T>` proporcionada por la biblioteca central de Cairo:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

El enum `Option<T>` es genérico sobre un tipo `T` y tiene dos variantes: `Some`, que contiene un valor de tipo `T`, y `None`, que no contiene ningún valor. Al utilizar el enum `Option<T>`, es posible expresar el concepto abstracto de un valor opcional y debido a que el valor tiene un tipo genérico `T`, podemos utilizar esta abstracción con cualquier tipo.

Los enums también pueden utilizar múltiples tipos genéricos, como la definición del enum `Result<T, E>` que proporciona la biblioteca estándar.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

El enum `Result<T, E>` tiene dos tipos genéricos, `T` y `E`, y dos variantes: `Ok` que tiene el valor de tipo `T` y `Err` que tiene el valor de tipo `E`. Esta definición hace que sea conveniente usar el enum `Result` en cualquier lugar donde tengamos una operación que pueda tener éxito (devolviendo un valor de tipo `T`) o fallar (devolviendo un valor de tipo `E`).

## Métodos Genéricos

También podemos implementar métodos en structs y enums, y usar los tipos genéricos en su definición. Utilizando nuestra definición anterior de la struct `Wallet<T>`, definimos un método `balance` para ella:

```rust
struct Wallet<T> {
    balance: T,
}

impl WalletDrop<T, impl TDrop: Drop<T>> of Drop<Wallet<T, U>>;

trait WalletTrait<T> {
    fn balance(self: @Wallet<T>) -> @T;
}

impl WalletImpl<T> of WalletTrait<T> {
    fn balance(self: @Wallet<T>) -> @T{
        return self.balance;
    }
}

fn main() {
    let w = Wallet {balance: 50};
    assert(w.balance() == 50, 0);
}
```

Primero definimos la clase `WalletTrait<T>` usando un tipo genérico `T` que define un método que devuelve una instantánea del campo `address` de `Wallet`. Luego, damos una implementación de la clase en `WalletImpl<T>`. Ten en cuenta que debes incluir un tipo genérico en ambas definiciones de la clase y la implementación.

También podemos especificar restricciones en los tipos genéricos al definir métodos en la clase. Por ejemplo, podríamos implementar métodos solo para instancias de `Wallet<u128>` en lugar de `Wallet<T>`. En el ejemplo de código, definimos una implementación para carteras que tienen un tipo concreto de `u128` para el campo `balance`.

```rust
trait WalletReceiveTrait {
    fn receive(ref self: Wallet<u128>, value: u128);
}

impl WalletReceiveImpl of WalletReceiveTrait {
    fn receive(ref self: Wallet<u128>, value: u128) {
        self.balance += value;
    }
}

fn main() {
    let mut w = Wallet {balance: 50_u128};
    assert(w.balance() == 50_u128, 0);

    w.receive(100_u128)
    assert(w.balance() == 150_u128, 0);
}
```

El nuevo método `receive` incrementa el tamaño del saldo de cualquier instancia de una `Wallet<u128>`. Observe que se cambió la función `main` haciendo que `w` sea una variable mutable para que pueda actualizar su saldo. Si cambiáramos la inicialización de `w` cambiando el tipo de `balance`, el código anterior no se compilaría.

Cairo nos permite definir métodos genéricos dentro de traits genéricos también. Usando la implementación previa de `Wallet<U, V>`, vamos a definir un trait que tome dos wallets de diferentes tipos genéricos y cree uno nuevo con un tipo genérico de cada uno. Primero, reescribamos la definición de la estructura:

```rust
struct Wallet<T, U> {
    balance: T,
    address: U,
}
```

A continuación vamos a definir de forma ingenua el trait y la implementación de `mixup`:

```rust
// This does not compile!
trait WalletMixTrait<T1, U1> {
    fn mixup<T2, U2>(self: Wallet<T1, U1>, other: Wallet<T2, U2>) -> Wallet<T1, U2>;
}

impl WalletMixImpl<T1,  U1> of WalletMixTrait<T1, U1> {
    fn mixup<T2, U2>(self: Wallet<T1, U1>, other: Wallet<T2, U2>) -> Wallet<T1, U2> {
        Wallet {balance: self.balance, address: other.address}
    }
}
```

Estamos creando un trait `WalletMixTrait<T1, U1>` con el método `mixup<T2, U2>` que, dada una instancia de `Wallet<T1, U1>` y `Wallet<T2, U2>`, crea un nuevo `Wallet<T1, U2>`. Como especifica la firma de `mixup`, tanto `self` como `other` se están eliminando al final de la función, lo que hace que este código no se compile. Si has estado siguiendo desde el principio hasta ahora, sabrás que debemos agregar un requisito para todos los tipos genéricos especificando que implementarán el trait `Drop` para que el compilador sepa cómo eliminar las instancias de `Wallet<T, U>`. La implementación actualizada es la siguiente:

```rust
trait WalletMixTrait<T1, U1> {
    fn mixup<T2, impl T2Drop: Drop<T2>, U2, impl U2Drop: Drop<U2>>(self: Wallet<T1, U1>, other: Wallet<T2, U2>) -> Wallet<T1, U2>;
}

impl WalletMixImpl<T1, impl T1Drop: Drop<T1>,  U1, impl U1Drop: Drop<U1>> of WalletMixTrait<T1, U1> {
    fn mixup<T2, impl T2Drop: Drop<T2>, U2, impl U2Drop: Drop<U2>>(self: Wallet<T1, U1>, other: Wallet<T2, U2>) -> Wallet<T1, U2> {
        Wallet {balance: self.balance, address: other.address}
    }
}
```

Sí, agregamos los requisitos para que `T1` y `U1` sean droppables en la declaración de `WalletMixImpl`. Luego hacemos lo mismo para `T2` y `U2`, esta vez como parte de la firma de `mixup`. Ahora podemos probar la función `mixup`:

```rs
fn main() {
   let w1 = Wallet{ balance: true, address: 10_u128};
   let w2 = Wallet{ balance: 32, address: 100_u8};

   let w3 = w1.mixup(w2);

   assert(w3.balance == true, 0);
   assert(w3.address == 100_u8, 0);
}
```

Primero creamos dos instancias: una de `Wallet<bool, u128>` y la otra de `Wallet<felt252, u8>`. Luego, llamamos a `mixup` y creamos una nueva instancia de `Wallet<bool, u8>`.

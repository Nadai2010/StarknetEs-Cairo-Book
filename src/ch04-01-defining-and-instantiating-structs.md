# Definiendo e instanciando una estructura

Las estructuras son similares a las tuplas, discutidas en la sección [Tipos de datos](ch02-02-data-types.md), en el sentido de que ambas contienen múltiples valores relacionados. Al igual que las tuplas, las piezas de una estructura pueden ser de diferentes tipos. A diferencia de las tuplas, en una estructura se nombra cada dato para que quede claro lo que significan los valores. Agregar estos nombres significa que las estructuras son más flexibles que las tuplas: no se tiene que depender del orden de los datos para especificar o acceder a los valores de una instancia.

Para definir una estructura, usamos la palabra reservada `struct` y nombramos la estructura completa. El nombre de una estructura debe describir la importancia de los datos que se agrupan. Luego, dentro de corchetes, definimos los nombres y tipos de los datos, a los que llamamos campos. Por ejemplo, el Listado 4-1 muestra una estructura que almacena información sobre una cuenta de usuario (*User*).


<span class="filename">Filename: structs.cairo</span>

```rust
#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}
```

<span class="caption">Listado 4-1: Definición de la estructura `User`</span>


Para usar una estructura después de haberla definido, creamos una instancia de esa estructura especificando valores concretos para cada uno de los campos. Creamos una instancia indicando el nombre de la estructura y luego agregamos corchetes que contienen pares de *clave: valor*, donde las claves son los nombres de los campos y los valores son los datos que queremos almacenar en esos campos. No tenemos que especificar los campos en el mismo orden en que los declaramos en la estructura. En otras palabras, la definición de una estructura es como una plantilla general para el tipo y las instancias completan esa plantilla con datos particulares para crear valores del tipo.

Por ejemplo, podemos declarar un usuario (*User*) en particular como se muestra en el Listado 4-2.

<span class="filename">Filename: structs.cairo</span>

```rust
#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}
fn main() {
    let user1 = User {
        active: true,
        username: 'someusername123',
        email: 'someone@example.com',
        sign_in_count: 1_u64,
    };
}
```

<span class="caption">Listado 4-2: Creando una instancia de la estructura `User`</span>

Para obtener un valor específico de una estructura, usamos la notación punto. Por ejemplo, para acceder a la dirección de correo electrónico de este usuario, usamos `user1.email`. Si la instancia es mutable, podemos cambiar un valor usando la notación punto y asignándolo a un campo en particular. El listado 4-3 muestra cómo cambiar el valor en el campo `email`de una instancia mutable de `User`.


<span class="filename">Filename: structs.cairo</span>

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: 'someusername123',
        email: 'someone@example.com',
        sign_in_count: 1_u64,
    };
    user1.email = 'anotheremail@example.com';
}
```

<span class="caption">Listado 4-3: Cambiando el valor del campo email de la instancia `User`</span>

Tenga en cuenta que toda la instancia debe ser mutable; Cairo no nos permite marcar solo ciertos campos como mutables.

Como con cualquier expresión, podemos construir una nueva instancia de la estructura como la última expresión en el cuerpo de la función para devolver implícitamente esa nueva instancia.

Listado 4-4 muestra la función `build_user` que retorna una instancia de la estructura `User` con el email y el username. Al campo `active` se le asigna el valor `true`,y el campo `sign_in_count` obtiene el valor de `1`.


<span class="filename">Filename: structs.cairo</span>

```rust
fn build_user(email: felt, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

<span class="caption">Listado 4-4: Función `build_user` que toma los argumentos *email* y *username*, y retorna una instancia de la estructura `User`</span>

Tiene sentido nombrar los parámetros de la función con el mismo nombre que los campos de la estructura, porque tener que repetir los nombres y variables de los campos `email`y `username` es un poco tedioso. Si la estructura tuviera más campos, repetir cada nombre sería aún más molesto. ¡Afortunadamente, hay una forma abreviada!


## Usando la abreviatura *Field Init*

Debido a que los nombres de los parámetros y los nombres de los campos de la estructura son exactamente iguales en el Listado 4-4, podemos usar la sintaxis abreviada de *Field Init* para reescribir la función `build_user` y que se comporte exactamente igual, pero no tenga la repetición de `username` y `email`, como se muestra en el Listado 4-5.

<span class="filename">Filename: structs.cairo</span>

```rust
fn build_user(email: felt252, username: felt252) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1_u64,
    }
}
```

<span class="caption">Listado 4-5: Función `build_user` que usa la abreviatura *field init* porque los parámetros `username` y `email` tienen el mismo nombre que los campos de la estructura</span>

Aquí, estamos creando una nueva instancia de la estructura `User`, que tiene un campo llamado `email`. Queremos establecer el valor del campo `email` con el valor del parámetro `email` de la función `build_user`. Debido a que el campo `email` y el parámetro `email` tienen el mismo nombre, solo necesitamos escribir `email` en lugar de `email: email`.
   

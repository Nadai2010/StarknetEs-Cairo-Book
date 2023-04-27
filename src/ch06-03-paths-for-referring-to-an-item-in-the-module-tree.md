## Caminos para hacer referencia a un elemento en el árbol de módulos

Para indicarle a Cairo dónde encontrar un elemento en el árbol de módulos, usamos un camino de la misma forma que usamos una ruta al navegar por un sistema de archivos. Para llamar a una función, necesitamos conocer su camino.

Un camino puede tomar dos formas:

- Un _camino absoluto_ es la ruta completa que comienza desde la raíz del crate. El camino absoluto comienza con el nombre del crate.
- Un _camino relativo_ comienza desde el módulo actual.

  Tanto los caminos absolutos como los relativos son seguidos por uno o más identificadores separados por dos puntos dobles (`::`).

Para ilustrar esta noción, tomemos de nuevo nuestro ejemplo del restaurante que usamos en el último capítulo. Tenemos un crate llamado `restaurant` en el cual tenemos un módulo llamado `front_of_house` que contiene un módulo llamado `hosting`. El módulo `hosting` contiene una función llamada `add_to_waitlist`. Queremos llamar a la función `add_to_waitlist` desde la función `eat_at_restaurant`. Necesitamos decirle a Cairo el camino hacia la función `add_to_waitlist` para que pueda encontrarla.

<span class="filename">Nombre del archivo: src/lib.cairo</span>

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}


pub fn eat_at_restaurant() {
    // Absolute path
    restaurant::front_of_house::hosting::add_to_waitlist(); // ✅ Compiles

    // Relative path
    front_of_house::hosting::add_to_waitlist(); // ✅ Compiles
}
```

La primera vez que llamamos a la función `add_to_waitlist` en `eat_at_restaurant`, usamos una ruta absoluta. La función `add_to_waitlist` está definida en la misma caja que `eat_at_restaurant`. En Cairo, las rutas absolutas comienzan desde la raíz de la caja, a la cual se refiere usando el nombre de la caja.

La segunda vez que llamamos a `add_to_waitlist`, usamos una ruta relativa. La ruta comienza con `front_of_house`, el nombre del módulo definido en el mismo nivel del árbol de módulos que `eat_at_restaurant`. Aquí, el equivalente en el sistema de archivos sería usar la ruta `./front_of_house/hosting/add_to_waitlist`. Comenzar con un nombre de módulo significa que la ruta es relativa al módulo actual.

### Comenzando rutas relativas con `super`

Elegir si usar o no `super` es una decisión que tomarás basada en tu proyecto y dependerá de si es más probable que muevas el código de definición de elementos por separado o junto con el código que usa el elemento.

<span class="filename">Nombre de archivo: src/lib.cairo</span>

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

Aquí se puede ver directamente que se accede fácilmente a un módulo padre usando `super`, lo que no era el caso anteriormente.

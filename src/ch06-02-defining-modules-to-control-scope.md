## Definición de módulos para controlar el ámbito

En esta sección, hablaremos sobre los módulos y otras partes del sistema de módulos, como las _rutas_ que le permiten nombrar elementos y la palabra clave `use` que introduce una ruta en el ámbito.

Primero, vamos a comenzar con una lista de reglas para su fácil referencia cuando esté organizando su código en el futuro. Luego explicaremos cada una de las reglas en detalle.

### Hoja de trucos de módulos

Aquí proporcionamos una referencia rápida sobre cómo funcionan los módulos, las rutas y la palabra clave `use` en el compilador, y cómo la mayoría de los desarrolladores organizan su código. Iremos a través de ejemplos de cada una de estas reglas a lo largo de este capítulo, pero este es un buen lugar para consultar como recordatorio de cómo funcionan los módulos. Puede crear un nuevo proyecto Scarb con `scarb new backyard` para seguir adelante.

- **Comience desde la raíz del crate**: Al compilar un crate, el compilador primero busca código para compilar en el archivo raíz del crate (_src/lib.cairo_).
- **Declaración de módulos**: En el archivo raíz del crate, puede declarar nuevos módulos; digamos que declara un módulo "garden" con `mod garden;`. El compilador buscará el código del módulo en estos lugares:

  - En línea, dentro de llaves que reemplazan al punto y coma que sigue a `mod garden;`.

    ```rust
      // crate root file (lib.cairo)
        mod garden {
        // code defining the garden module goes here
        }
    ```

- En el archivo _src/garden.cairo_
- **Declarando submódulos**: En cualquier archivo que no sea la raíz del paquete, puede declarar submódulos. Por ejemplo, podría declarar `mod vegetables;` en el archivo _src/garden.cairo_. El compilador buscará el código del submódulo dentro del directorio nombrado por el módulo padre en estos lugares:

  - En línea, directamente después de `mod vegetables`, dentro de llaves en lugar del punto y coma.
    ```rust
    // src/garden.cairo file
    mod vegetables {
        // code defining the vegetables submodule goes here
    }
    ```

  - En el archivo _src/garden/vegetables.cairo_

- **Rutas a código en módulos**: Una vez que un módulo forma parte de su paquete, puede hacer referencia al código de ese módulo desde cualquier otro lugar en ese mismo paquete, utilizando la ruta al código. Por ejemplo, un tipo `Asparagus` en el módulo de vegetales del jardín se encontraría en `backyard::garden::vegetables::Asparagus`.
- **La palabra clave `use`**: Dentro de un alcance, la palabra clave `use` crea atajos a elementos para reducir la repetición de rutas largas. En cualquier alcance que pueda hacer referencia a `backyard::garden::vegetables::Asparagus`, puede crear un atajo con `use backyard::garden::vegetables::Asparagus;` y a partir de entonces solo necesita escribir `Asparagus` para usar ese tipo en el alcance.

Aquí creamos un paquete llamado `backyard` que ilustra estas reglas. El directorio del paquete, también llamado `backyard`, contiene estos archivos y directorios:

```text
backyard/
├── Scarb.toml
├── cairo_project.toml
└── src
    ├── garden
    │   └── vegetables.cairo
    ├── garden.cairo
    └── lib.cairo
```

> Nota: Aquí se observa un archivo `cairo_project.toml`.
> Este es el archivo de configuración para proyectos "vanilla" de Cairo (es decir, no 
> gestionados por Scarb), que se requiere para ejecutar el comando `cairo-run .` y 
> ejecutar el código del crate.
> Es necesario hasta que Scarb implemente esta función. El contenido del archivo es:
>
> ```toml
> [crate_roots]
> backyard = "src"
> ```
>
> y indica que la caja llamada "backyard" se encuentra en el directorio `src`.

El archivo raíz de la caja en este caso es _src/lib.cairo_, y contiene:

<span class="filename">Nombre de archivo: src/lib.cairo</span>

```rust
use garden::vegetables::Asparagus;

mod garden;

fn main(){
    let Asparagus = Asparagus{};
}


```

La línea `mod garden;` le indica al compilador que incluya el código que encuentra en _src/garden.cairo_, que es:

<span class="filename">Nombre del archivo: src/garden.cairo</span>

```rust
mod vegetables;
```

Aquí, `mod vegetables;` significa que el código en _src/garden/vegetables.cairo_ también está incluido. Ese código es:

```rust
#[derive(Copy,Drop)]
struct Asparagus{}
```

La línea `use garden::vegetables::Asparagus;` nos permite traer el tipo `Asparagus` al ámbito de alcance, para que podamos usarlo en la función `main`.

¡Ahora vamos a entrar en los detalles de estas reglas y demostrarlas en acción!

### Agrupando el Código Relacionado en Módulos

_Los módulos_ nos permiten organizar el código dentro de un paquete para hacerlo más legible y fácil de reutilizar. Como ejemplo, escribiremos un paquete de biblioteca que proporcione la funcionalidad de un restaurante. Definiremos las firmas de las funciones pero dejaremos sus cuerpos vacíos para concentrarnos en la organización del código, en lugar de en la implementación de un restaurante.

En la industria de la restauración, algunas partes de un restaurante se denominan _front of house_ (delante de la casa) y otras como _back of house_ (detrás de la casa). Front of house es donde están los clientes; esto abarca desde donde los anfitriones sientan a los clientes, los servidores toman órdenes y pagos, y los barman hacen bebidas. Back of house es donde los chefs y cocineros trabajan en la cocina, los lavaplatos limpian y los gerentes hacen trabajo administrativo.

Para estructurar nuestro paquete de esta manera, podemos organizar sus funciones en módulos anidados. Cree un nuevo paquete llamado `restaurant` ejecutando el comando `scarb new restaurant`; luego ingrese el código en el Listado 6-1 en _src/lib.cairo_ para definir algunos módulos y firmas de funciones. Aquí está la sección de front of house: 

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
```

<span class="caption">Listado 6-1: Un módulo `front_of_house` que contiene otros módulos que a su vez contienen funciones</span>

Definimos un módulo con la palabra clave `mod` seguida del nombre del módulo (en este caso, `front_of_house`). El cuerpo del módulo va entre llaves. Dentro de los módulos, podemos colocar otros módulos, como en este caso con los módulos `hosting` y `serving`. Los módulos también pueden contener definiciones de otros elementos, como structs, enums, constantes, traits y, como en el Listado 6-1, funciones.

Al utilizar módulos, podemos agrupar las definiciones relacionadas y darles un nombre que indique por qué están relacionadas. Los programadores que usan este código pueden navegar por el código en función de los grupos en lugar de tener que leer todas las definiciones, lo que hace que sea más fácil encontrar las definiciones relevantes para ellos. Los programadores que agregan nueva funcionalidad a este código sabrían dónde colocar el código para mantener el programa organizado.

Anteriormente, mencionamos que _src/lib.cairo_ se llama raíz de la caja. La razón de este nombre es que el contenido de este archivo forma un módulo con el nombre de la caja en la raíz de la estructura de módulos de la caja, conocido como el _árbol de módulos_.

El Listado 6-2 muestra el árbol de módulos para la estructura en el Listado 6-1.

```text
restaurant
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

<span class="caption">Listing 6-2: El árbol de módulos para el código en el Listado
6-1</span>

Este árbol muestra cómo algunos módulos se anidan dentro de otros; por ejemplo,
`hosting` se anida dentro de `front_of_house`. El árbol también muestra que algunos módulos
son _hermanos_ entre sí, lo que significa que están definidos en el mismo módulo;
`hosting` y `serving` son hermanos definidos dentro de `front_of_house`. Si el módulo
A está contenido dentro del módulo B, decimos que el módulo A es el _hijo_ del módulo B
y que el módulo B es el _padre_ del módulo A. Observa que todo el árbol de módulos está enraizado en el nombre explícito del paquete `restaurant`.

El árbol de módulos podría recordarte al árbol de directorios del sistema de archivos en tu computadora; ¡esta es una comparación muy adecuada! Al igual que los directorios en un sistema de archivos, utilizamos los módulos para organizar nuestro código. Y al igual que los archivos en un directorio, necesitamos una manera de encontrar nuestros módulos.
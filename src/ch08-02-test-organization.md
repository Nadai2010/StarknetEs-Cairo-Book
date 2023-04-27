# Organización de Test

Pensaremos en las pruebas en términos de dos categorías principales: pruebas unitarias y pruebas de integración. Las pruebas unitarias son pequeñas y más enfocadas, probando un módulo a la vez en aislamiento, y pueden probar funciones privadas. Las pruebas de integración utilizan su código de la misma manera que cualquier otro código externo, utilizando solo la interfaz pública y potencialmente ejercitando varios módulos por prueba.

Escribir ambos tipos de pruebas es importante para asegurarse de que las piezas de su biblioteca estén haciendo lo que se espera de ellas, tanto separadas como juntas.

## Test Unitarios

El propósito de las pruebas unitarias es probar cada unidad de código en aislamiento del resto del código para identificar rápidamente dónde el código funciona y dónde no lo hace como se esperaba. Colocará las pruebas unitarias en el directorio `src` en cada archivo con el código que están probando.

La convención es crear un módulo llamado `tests` en cada archivo para contener las funciones de prueba y anotar el módulo con `cfg(test)`.

### El Módulo de Test y `#[cfg(test)]`

La anotación `#[cfg(test)]` en el módulo de pruebas indica a Cairo que compile y ejecute el código de prueba solo cuando se ejecuta `cairo-test`, no cuando se ejecuta `cairo-run`. Esto ahorra tiempo de compilación cuando solo desea compilar la biblioteca y ahorra espacio en el artefacto compilado resultante porque las pruebas no están incluidas. Verá que debido a que las pruebas de integración van en un directorio diferente, no necesitan la anotación `#[cfg(test)]`. Sin embargo, debido a que las pruebas unitarias van en los mismos archivos que el código, usará `#[cfg(test)]` para especificar que no deben incluirse en el resultado compilado.

Recuerde que cuando creamos el nuevo proyecto `adder` en la primera sección de este capítulo, escribimos esta primera prueba:

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

El atributo `cfg` significa "configuración" y le indica a Cairo que el siguiente elemento solo debe incluirse dado una cierta opción de configuración. En este caso, la opción de configuración es `test`, que es proporcionada por Cairo para compilar y ejecutar pruebas. Al usar el atributo `cfg`, Cairo compila nuestro código de prueba solo si ejecutamos activamente las pruebas con `cairo-test`. Esto incluye cualquier función de ayuda que pueda estar dentro de este módulo, además de las funciones anotadas con `#[test]`.

## Test de Integración

Las pruebas de integración usan su biblioteca de la misma manera que cualquier otro código. Su propósito es probar si muchas partes de su biblioteca funcionan correctamente juntas. Las unidades de código que funcionan correctamente por sí mismas podrían tener problemas cuando se integran, por lo que también es importante tener cobertura de prueba del código integrado. Para crear pruebas de integración, primero necesita un directorio de `tests`.

### Directorio `tests`

```shell
adder
├── cairo_project.toml
├── src
    ├── lib.cairo
│   └── main.cairo
└── tests
    ├── lib.cairo
    └── integration_test.cairo
```

<!-- TODO: eliminar cuando las pruebas de Scarab funcionen -->

> Para ejecutar correctamente tus pruebas con `cairo-test`, deberás actualizar tu archivo `cairo_project.toml` para agregar la declaración de tu crate `tests`.
>
> ```rust
> [crate_roots]
> adder = "src"
> tests = "tests"
> ```

Cada archivo de prueba se compila como una entidad separada, por eso cada vez que agregas un nuevo archivo de prueba debes agregarlo a tu archivo _tests/lib.cairo_.

<span class="filename">Nombre de archivo: tests/lib.cairo</span>

```rust
#[cfg(tests)]
mod integration_tests;
```

Ingrese el código del Listado 11-13 en el archivo _tests/integration_test.cairo_:

<span class="filename">Nombre de archivo: tests/integration_test.cairo</span>

```rust
use adder::main;

#[test]
fn internal() {
    assert(main::internal_adder(2_u32, 2_u32) == 4_u32, 'internal_adder failed');
}
```

Cada archivo en el directorio de pruebas es una creación separada, por lo que debemos incluir nuestra biblioteca en el alcance de cada creación de prueba. Por esa razón, agregamos `use adder::main` en la parte superior del código, lo cual no necesitábamos en las pruebas unitarias.

```shell
$ cairo-test tests/
running 1 tests
test tests::tests_integration::it_adds_two ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

El resultado de las pruebas es el mismo que hemos estado viendo: una línea por cada prueba.

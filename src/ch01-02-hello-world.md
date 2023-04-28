## Hola, Mundo

Ahora que has instalado Cairo, es hora de escribir tu primer programa Cairo.
Es tradicional cuando se aprende un nuevo lenguaje escribir un pequeño programa que
imprima el texto `¡Hola, mundo!` en la pantalla, ¡así que haremos lo mismo aquí!

> Nota: Este libro asume una familiaridad básica con la línea de comandos. Cairo no hace
> ninguna demanda específica sobre su edición o herramientas o donde vive su código, así que
> si prefiere usar un entorno de desarrollo integrado (IDE) en lugar de la línea de comandos, > siéntase libre de hacerlo.
> la línea de comandos, siéntase libre de usar su IDE favorito. El equipo de Cairo ha desarrollado
> una extensión VSCode para el lenguaje Cairo que puedes usar para obtener las características de
> el servidor de lenguajes y el resaltado de código. Ver [Apéndice A][devtools]<!-- ignore -->
> para más detalles.

### Creando un Directorio de Proyecto

Empezarás creando un directorio para almacenar tu código de Cairo. A Cairo no le importa
a Cairo dónde vive tu código, pero para los ejercicios y proyectos de este libro,
sugerimos hacer un directorio _cairo_projects_ en su directorio home y mantener todos
tus proyectos allí.

Abre un terminal e introduce los siguientes comandos para crear un directorio _cairo_projects_.
y un directorio para el proyecto "¡Hola, mundo!" dentro del directorio _cairo_projects_.

Para Linux, macOS y PowerShell en Windows, introduce esto:

```console
mkdir ~/cairo_projects
cd ~/cairo_projects
mkdir hello_world
cd hello_world
```

Para Windows CMD, introduzca esto:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Escribiendo y Ejecutando un Programa Cairo

A continuación, crea un nuevo archivo fuente y llámalo _main.cairo_. Los archivos Cairo siempre terminan con la extensión _.cairo_. Si usas más de una palabra en tu nombre de archivo, la convención es usar un guión bajo para separarlas. Por ejemplo_, hello_mundo.cairo_ en lugar de _helloworld.cairo_.

Ahora abre el archivo _main.cairo_ que acabas de crear e introduce el código del Listado 1-1.

<span class="filename">Filename: main.cairo</span>

```rust
use debug::PrintTrait;
fn main() {
    'Hello, world!'.print();
}
```

<span class="caption">Listing 1-1: A program that prints `Hello, world!`</span>

Guarde el archivo y vuelva a su ventana de terminal en el directorio
_~/cairo_projects/hello_world_. Introduzca los siguientes comandos
para compilar y ejecutar el archivo:

```console
$ cairo-run main.cairo
Hello, world!
```

Independientemente de su sistema operativo, la cadena `Hello, world!` debería imprimirse en el terminal.

Si `Hello, world!` se imprime, ¡enhorabuena! Has escrito oficialmente un programa en Cairo. Eso te convierte en un programador de Cairo ¡Bienvenido!

### Anatomía de un Programa Cairo

Revisemos este programa "¡Hola, mundo!" en detalle. Aquí está la primera pieza del puzzle:

```rust
fn main() {

}
```
Estas líneas definen una función llamada `main`. La función `main` es especial: es
es siempre el primer código que se ejecuta en cada programa ejecutable de El Cairo. Aquí, la
primera línea declara una función llamada `main` que no tiene parámetros y devuelve
nada. Si hubiera parámetros, irían dentro de los paréntesis `()`.

El cuerpo de la función está envuelto en `{}`. Cairo requiere llaves alrededor de todos los
cuerpos de función. Es de buen estilo colocar la llave de apertura en la misma línea que la declaración de la función, añadiendo un espacio en medio.

> Nota: Si quieres mantener un estilo estándar en todos los proyectos de Cairo, puedes
> usar la herramienta de formateo automático llamada `cairo-format` para formatear tu código 
> en un estilo particular (más sobre `cairo-format` en
> [Apéndice A][devtools]<!-- ignore -->). El equipo de Cairo ha incluido esta herramienta
> con la distribución estándar de Cairo, como lo es `cairo-run`, así que ya debería estar
> ¡instalada en tu ordenador!

Antes de la declaración de la función principal, la línea `use debug::PrintTrait;` es responsable de importar un elemento definido en otro módulo. En este caso, estamos importando el elemento `PrintTrait` de la biblioteca central de Cairo. Haciendo esto, ganamos la habilidad de usar el método `print()` en tipos de datos que son compatibles con la impresión.

El cuerpo de la función `main` contiene el siguiente código:

```rust
    'Hello, world!'.print();
```

Esta línea hace todo el trabajo en este pequeño programa: imprime texto en la pantalla.
pantalla. Hay cuatro detalles importantes a tener en cuenta aquí.

En primer lugar, el estilo de Cairo es hacer sangrías con cuatro espacios, no con una tabulación.

Segundo, la función `print()` es un método del trait `PrintTrait`. Este trait se importa de la librería del núcleo de Cairo, y define cómo imprimir valores en la pantalla para diferentes tipos de datos. En nuestro caso, nuestro texto está definido como una "cadena corta", que es una cadena ASCII que puede caber en el tipo de datos básico de Cairo, que es el tipo `felt252`. Al llamar a `Hello, world!'.print()`, estamos llamando al método `print()` de la implementación `felt252` del trait `PrintTrait`.

En tercer lugar, ves la cadena corta `'Hello, world!'` Pasamos esta cadena corta como argumento
a `print()`, y la cadena corta se imprime en la pantalla.

Cuarto, terminamos la línea con un punto y coma (`;`), que indica que esta
expresión ha terminado y la siguiente está lista para comenzar. La mayoría de las líneas de código de Cairo terminan con punto y coma.

Sólo ejecutar con `cairo-run` está bien para programas simples, pero a medida que tu proyecto
proyecto crezca, querrá manejar todas las opciones y hacer fácil compartir su código.
código. A continuación, te presentaremos la herramienta Scarb, que te ayudará a escribir
programas Cairo del mundo real.

[devtools]: appendix-04-useful-development-tools.md

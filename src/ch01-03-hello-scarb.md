# Hola, Scarb

Scarb es el gestor de paquetes de Cairo y está fuertemente inspirado en [Cargo](https://doc.rust-lang.org/cargo/), el sistema de construcción y gestor de paquetes de Rust.

Scarb maneja muchas tareas por ti, como construir tu código (ya sea Cairo puro o contratos Starknet), descargar las librerías de las que depende tu código, y construir esas librerías.

Si fuéramos a construir el proyecto `Hello, world!` usando Scarb, sólo la parte de Scarb que maneja la construcción del código sería utilizada, ya que el programa no requiere ninguna dependencia externa. A medida que escriba programas Cairo más complejos, agregará dependencias, y si inicia un proyecto utilizando Scarb, agregar dependencias será mucho más fácil de hacer.

Empecemos instalando Scarb.

## Instalando Scarb

### Requisitos

Scarb requiere un ejecutable Git disponible en la variable de entorno `PATH`.

### Instalación

Por ahora, Scarb necesita instalación manual con los siguientes pasos:

- Descargue el archivo de versiones correspondiente a su sistema operativo y arquitectura de CPU desde [Scarb releases on GitHub](https://github.com/software-mansion/scarb/releases)
- Extráigalo a una ubicación en la que desee tener Scarb instalado, por ejemplo. `~/scarb`
- Añada la ruta al directorio `scarb/bin` a su variable de entorno `PATH`.

  Esto depende del shell que esté utilizando. Tomemos el ejemplo de [zsh](https://ohmyz.sh/) y has extraido Scarb a `~/scarb`:

  - Abra el archivo `~/.zshrc` en su editor favorito
  -Añada la siguiente línea al final del archivo: `export PATH="$PATH:~/scarb/bin"`

- Verifique la instalación ejecutando el siguiente comando en una nueva sesión de terminal, debería imprimir las versiones en Scarb y Cairo, por ejemplo:

  ```bash
  $ scarb --version
  scarb 0.1.0 (289137c24 2023-03-28)
  cairo: 1.0.0-alpha.6
  ```

### Creando un Proyecto con Scarb

Vamos a crear un nuevo proyecto utilizando Scarb y ver en qué se diferencia de nuestro proyecto original “Hello, world!”.

Navegue de nuevo a su directorio de proyectos (o donde haya decidido almacenar su código). Luego ejecute lo siguiente:

```bash
$ scarb new hello_scarb
```

Crea un nuevo directorio y proyecto llamado hello_scarb. Hemos llamado a nuestro proyecto hello_scarb, y Scarb crea sus archivos en un directorio con el mismo nombre.

Entre en el directorio hello_scarb con el comando `cd hello_scarb`.Verás que Scarb ha generado dos archivos y un directorio para nosotros: un archivo `Scarb.toml` y un directorio src con un archivo `lib.cairo` dentro.

También ha inicializado un nuevo repositorio Git junto con un archivo `.gitignore`.

> Nota: Git es un sistema de control de versiones común. Puede dejar de usar el sistema de control de versiones 
> usando la bandera `--vcs`.
> Ejecute `scarb new -help` para ver las opciones disponibles.

Abra _Scarb.toml_ en su editor de texto preferido. Debería parecerse al código del Listado 1-2.

<span class="filename">Filename: Scarb.toml</span>

```toml
[package]
name = "hello_scarb"
version = "0.1.0"

# Vea más claves y sus definiciones en https://docs.swmansion.com/scarb/docs/reference/manifest

[dependencies]
# foo = { path = "vendor/foo" }
```

<span class="caption">Listing 1-2: Contents of Scarb.toml generated by `scarb new`</span>

Este archivo se encuentra en formato [TOML](https://toml.io/) (Tom’s Obvious, Minimal Language), que es el formato de configuración de Scarb.

La primera línea, `[package]`, es un encabezado de sección que indica que las siguientes sentencias están configurando un paquete. A medida que agreguemos más información a este archivo, agregaremos otras secciones.

Las siguientes dos líneas establecen la información de configuración que Scarb necesita para compilar su programa: el nombre y la versión de Scarb a utilizar.

La última línea, `[dependencies]`, es el comienzo de una sección para que puedas listar cualquiera de las dependencias de tu proyecto. En Cairo, los paquetes de código se conocen como crates. No necesitaremos ninguna otra crate para este proyecto.

El otro archivo creado por Scarb es `src/lib.cairo`, borremos todo el contenido y pongamos el siguiente contenido, explicaremos la razón más adelante.

```rust
mod hello_scarb;
```

A continuación, cree un nuevo archivo llamado `src/hello_scarb.cairo` y ponle el siguiente código:

<span class="filename">Filename: src/hello_scarb.cairo</span>

```rust
use debug::PrintTrait;
fn main() {
    'Hello, Scarb!'.print();
}
```
Acabamos de crear un archivo llamado `lib.cairo`, que contiene una declaración de módulo que hace referencia a otro módulo llamado "hello_scarb", así como el archivo `hello_scarb.cairo`, que contiene los detalles de implementación del módulo "hello_scarb".

Scarb requiere que sus archivos fuente se encuentren dentro del directorio src.

El directorio de proyecto de nivel superior está reservado para los archivos README, información de licencia, archivos de configuración, y cualquier otro contenido no relacionado con el código.
Scarb asegura una ubicación designada para todos los componentes del proyecto, manteniendo una organización estructurada.

Si ha iniciado un proyecto que no utiliza Scarb, como hicimos con el proyecto “Hello, world!”, puede convertirlo en un proyecto que utilice Scarb. Mueva el código del proyecto al directorio src y cree un archivo `Scarb.toml` apropiado.

### Creación de un proyecto Scarb

Desde su directorio hello_scarb, construya su proyecto introduciendo el siguiente comando:

```bash
$ scarb build
   Compiling hello_scarb v0.1.0 (file:///projects/Scarb.toml)
    Finished release target(s) in 0 seconds
```

Este comando crea un archivo `sierra` en `target/release`, ignoremos el archivo `sierra` por ahora.

Si has instalado Cairo correctamente, deberías ser capaz de ejecutarlo y ver la siguiente salida:

```bash
$ cairo-run src/lib.cairo
[DEBUG] Hello, Scarb!                   (raw: 5735816763073854913753904210465)

Run completed successfully, returning []
```
> Nota: Notarás aquí que no usamos un comando de Scarb, sino un comando de los binarios de Cairo directamente.
> Como Scarb no tiene un comando para ejecutar código de Cairo, tenemos que usar el comando `cairo-run` directamente.
> Usaremos este comando en el resto del tutorial, pero también usaremos comandos de Scarb para inicializar proyectos.

### Definición de scripts personalizados

Podemos definir scripts scarb en el archivo `Scarb.toml`, que puede ser usado para ejecutar scripts shell personalizados.

Añada la siguiente línea a su fichero `Scarb.toml`:

```toml
[scripts]
run-lib = "cairo-run src/lib.cairo"
```

Ahora puede ejecutar el siguiente comando para ejecutar el proyecto:

```bash
$ scarb run run-lib
[DEBUG] Hello, Scarb!                   (raw: 5735816763073854913753904210465)

Run completed successfully, returning []
```

Usar `scarb run` es una forma conveniente de ejecutar scripts de shell personalizados que pueden ser útiles para ejecutar archivos y probar su proyecto.

Recapitulemos lo que hemos aprendido hasta ahora sobre Scarb:

- Podemos crear un proyecto usando `scarb new`.
- Podemos construir un proyecto usando `scarb build` para generar el código Sierra compilado.
- Podemos definir scripts personalizados en `Scarb.toml` y llamarlos con el comando `scarb run`.

Una ventaja adicional de usar Scarb es que los comandos son los mismos sin importar el sistema operativo en el que estemos trabajando. Así que, en este punto, ya no proporcionaremos instrucciones específicas para Linux y macOS frente a Windows.
# Resumen

Ya has empezado con buen pie tu viaje en Cairo. En este capítulo, has aprendido cómo:

- Instalar la última versión estable de Cairo
- Escribir y ejecutar un programa " Hello, world!" usando `cairo-run` directamente
- Crear y ejecutar un nuevo proyecto usando las convenciones de Scarb

Este es un buen momento para construir un programa más sustancial para acostumbrarte a leer y escribir código de Cairo.
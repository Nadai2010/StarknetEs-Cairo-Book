# Paquetes y Crates

## ¿Qué es un Crate?
Un crate es la cantidad más pequeña de código que el compilador de Cairo considera a la vez. Incluso si ejecuta `cairo-compile` en lugar de `scarb build` y pasa un solo archivo de código fuente, el compilador considera que ese archivo es un crate. Los crates pueden contener módulos, y los módulos pueden estar definidos en otros archivos que se compilan junto con el crate, como se discutirá en las secciones siguientes.

## ¿Qué es la Raíz del Crate?
La raíz del crate es el archivo de origen `lib.cairo` desde el cual el compilador de Cairo comienza y forma el módulo raíz de su crate (explicaremos los módulos en profundidad en la sección ["Definición de módulos para controlar el alcance"](./ch06-02-defining-modules-to-control-scope.md)).

## ¿Qué es un Paquete?
Un paquete de Cairo es un conjunto de uno o más crates con un archivo Scarb.toml que describe cómo construir esos crates. Esto permite la división del código en partes más pequeñas y reutilizables, y facilita la gestión de dependencias más estructurada.

## Creación de un Paquete con Scarb
Puede crear un nuevo paquete de Cairo utilizando la herramienta de línea de comandos scarb. Para crear un nuevo paquete, ejecute el siguiente comando: "
```bash
scarb new my_crate
```

Este comando generará un nuevo directorio de paquete llamado my_crate con la siguiente estructura:
```
my_crate/
├── Scarb.toml
└── src
    └── lib.cairo
```

- `src/` es el directorio principal donde se almacenarán todos los archivos de origen de Cairo para el paquete.
- `lib.cairo` es el módulo raíz predeterminado del crate, que también es el punto de entrada principal del paquete. Por defecto, está vacío.
- `Scarb.toml` es el archivo de manifiesto del paquete, que contiene metadatos y opciones de configuración para el paquete, como dependencias, nombre del paquete, versión y autores. Puede encontrar documentación al respecto en la [referencia de Scarb](https://docs.swmansion.com/scarb/docs/reference/manifest).

```toml
[package]
name = "my_crate"
version = "0.1.0"

[dependencies]
# foo = { path = "vendor/foo" }
```

A medida que desarrolla su paquete, es posible que desee organizar su código en varios archivos de origen de Cairo. Puede hacer esto creando archivos `.cairo` adicionales dentro del directorio `src` o sus subdirectorios.
# Tipos Genéricos y Traits

Cada lenguaje de programación tiene herramientas para manejar eficazmente la duplicación de conceptos. En Cairo, una de esas herramientas son los genéricos: sustitutos abstractos de tipos concretos u otras propiedades. Podemos expresar el comportamiento de los genéricos o cómo se relacionan con otros genéricos sin saber qué habrá en su lugar al compilar y ejecutar el código.

Las funciones, estructuras, enumeraciones y traits pueden incorporar tipos genéricos como parte de su definición en lugar de tipos concretos como `u32` o `ContractAddress`.

Los genéricos nos permiten reemplazar tipos específicos con un marcador de posición que representa múltiples tipos para eliminar la duplicación de código.

Para cada tipo concreto que reemplaza a un tipo genérico, el compilador crea una nueva definición, reduciendo el tiempo de desarrollo para el programador, pero la duplicación de código a nivel de compilación todavía existe. Esto puede ser importante si estás escribiendo contratos Starknet y usando un genérico para múltiples tipos que hará que el tamaño del contrato aumente.

Luego aprenderás cómo usar traits para definir comportamientos de manera genérica. Puedes combinar traits con tipos genéricos para restringir un tipo genérico para que acepte solo aquellos tipos que tienen un comportamiento particular, en lugar de cualquier tipo.
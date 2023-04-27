# Gestión de proyectos Cairo con Paquetes, Crates y Módulos

A medida que escriba programas grandes, la organización de su código se volverá cada vez más importante. Al agrupar funcionalidades relacionadas y separar el código con características distintas, aclarará dónde encontrar el código que implementa una característica en particular y dónde ir para cambiar cómo funciona una característica.

Los programas que hemos escrito hasta ahora han estado en un módulo en un solo archivo. A medida que un proyecto crece, debe organizar el código dividiéndolo en múltiples módulos y luego en múltiples archivos. A medida que un paquete crece, puede extraer partes en Crates separados que se convierten en dependencias externas. Este capítulo cubre todas estas técnicas.

También discutiremos la encapsulación de detalles de implementación, lo que le permite reutilizar el código a un nivel superior: una vez que ha implementado una operación, otro código puede llamar a su código sin tener que saber cómo funciona la implementación.

Un concepto relacionado es el ámbito: el contexto anidado en el que se escribe el código tiene un conjunto de nombres que se definen como "en ámbito". Al leer, escribir y compilar código, los programadores y compiladores deben saber si un nombre particular en un lugar particular se refiere a una variable, función, estructura, enumeración, módulo, constante u otro elemento y qué significa ese elemento. Puede crear ámbitos y cambiar qué nombres están dentro o fuera de ámbito. No puede tener dos elementos con el mismo nombre en el mismo ámbito.

Cairo tiene varias características que le permiten gestionar la organización de su código. Estas características, a veces denominadas colectivamente el _sistema de módulos_, incluyen:

- **Paquetes:** Una característica de Scarb que le permite construir, probar y compartir Crates.
- **Crates:** Un árbol de módulos que corresponde a una única unidad de compilación. Tiene un directorio raíz y un módulo raíz definido en el archivo `lib.cairo` bajo este directorio.
- **Módulos** y **use:** le permiten controlar la organización y el ámbito de los elementos.
- **Rutas:** una forma de nombrar un elemento, como una estructura, función o módulo.

En este capítulo, cubriremos todas estas características, discutiremos cómo interactúan y explicaremos cómo usarlas para gestionar el ámbito. Al final, debería tener una comprensión sólida del sistema de módulos y ser capaz de trabajar con ámbitos como un profesional.
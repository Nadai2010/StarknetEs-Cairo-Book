## Apéndice A - Herramientas de desarrollo útiles

En este apéndice, hablamos de algunas herramientas de desarrollo útiles que el proyecto Cairo
proporciona. Veremos el formateo automático, formas rápidas de aplicar
correcciones de advertencias, un linter, y la integración con IDEs.

### Formateo automático con `cairo-format`.

La herramienta `cairo-format` reformatea tu código de acuerdo con el estilo de código de la comunidad. Muchos proyectos colaborativos usan `cairo-format` para evitar discusiones sobre qué estilo usar al escribir Cairo: todo el mundo formatea su código usando la herramienta.

Para formatear cualquier proyecto de Cairo, introduce lo siguiente:

```console
cairo-format -r
```
Ejecutando este comando reformateará todo el código de Cairo en el directorio actual de forma recursiva. Esto solo cambiará el estilo de código, no la semántica del código.

### Integración del IDE usando `cairo-language-server`

Para ayudar con la integración del IDE, la comunidad de Cairo recomienda el uso de [`cairo-language-server`](https://github.com/starkware-libs/cairo/tree/main/crates/cairo-lang-language-server). Esta herramienta es un conjunto de utilidades centradas en el compilador que utiliza el [Protocolo del Servidor de Lenguaje](http://langserver.org/), que es una especificación para que los IDE y los lenguajes de programación se comuniquen entre sí. Diferentes clientes pueden utilizar `cairo-language-server`, como la extensión de Cairo para Visual Studio Code.

Visita la [página](https://github.com/starkware-libs/cairo/tree/main/vscode-cairo) de `vscode-cairo` para obtener instrucciones de instalación. Obtendrás habilidades como autocompletado, saltar a la definición y errores en línea.

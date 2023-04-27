# Introducción

## ¿Qué es Cairo?

Cairo es un lenguaje de programación diseñado para una CPU virtual del mismo nombre. El aspecto único de este procesador es que no fue creado para las restricciones físicas de nuestro mundo, sino para las criptográficas, lo que lo hace capaz de probar eficientemente la ejecución de cualquier programa que se ejecute en él. Esto significa que puedes realizar operaciones que consumen mucho tiempo en una máquina en la que no confías, y comprobar el resultado muy rápidamente en una máquina más barata.

Mientras que Cairo 0 solía compilarse directamente a CASM, el ensamblador de CPU de Cairo, Cairo 1 es un lenguaje de más alto nivel. Primero compila a Sierra, una representación intermedia de Cairo que compilará más tarde a un subconjunto seguro de CASM. El objetivo de Sierra es garantizar que tu CASM sea siempre demostrable, incluso cuando falle el cálculo.

## ¿Qué puedes hacer con Cairo?

Cairo te permite calcular valores confiables en máquinas no confiables. Un caso de uso importante es Starknet, una solución para la escalabilidad de Ethereum. Ethereum es una plataforma blockchain descentralizada que permite la creación de aplicaciones descentralizadas donde cada interacción entre un usuario y una d-app es verificada por todos los participantes. Starknet es una capa 2 construida sobre Ethereum. En lugar de que todos los participantes de la red verifiquen todas las interacciones del usuario, solo un nodo, llamado el probador, ejecuta los programas y genera pruebas de que los cálculos se realizaron correctamente. Estas pruebas luego son verificadas por un contrato inteligente de Ethereum, lo que requiere significativamente menos potencia de cálculo en comparación con la ejecución de las interacciones mismas. Este enfoque permite un mayor rendimiento y una reducción en los costos de transacción, pero preservando la seguridad de Ethereum.

## ¿Cuáles son las diferencias con otros lenguajes de programación?

Cairo es bastante diferente de los lenguajes de programación tradicionales, especialmente en cuanto a los costos generales y sus ventajas principales. Tu programa se puede ejecutar de dos formas diferentes:

- Cuando se ejecuta por el probador, es similar a cualquier otro lenguaje. Debido a que Cairo está virtualizado y porque las operaciones no se diseñaron específicamente para la máxima eficiencia, esto puede llevar a una sobrecarga de rendimiento, pero no es la parte más relevante para optimizar.

- Cuando el probador genera la prueba, es un poco diferente. Esto tiene que ser lo más barato posible, ya que potencialmente se podría verificar en muchas máquinas muy pequeñas. Afortunadamente, verificar es más rápido que computar y Cairo tiene algunas ventajas únicas para mejorar aún más. Uno notable es la no determinación. Este es un tema que cubrirás con más detalle más adelante en este libro, pero la idea es que teóricamente puedes usar un algoritmo diferente para verificar que para calcular. Actualmente, escribir código personalizado no determinista no está soportado para los desarrolladores, pero la biblioteca estándar aprovecha la no determinación para mejorar el rendimiento. Por ejemplo, ordenar una matriz en Cairo cuesta lo mismo que copiarla. Debido a que el verificador no ordena la matriz, simplemente verifica que está ordenada, lo que es más barato.

Otro aspecto que diferencia al lenguaje es su modelo de memoria. En Cairo, el acceso a la memoria es inmutable, lo que significa que una vez que se escribe un valor en la memoria, no se puede cambiar. Cairo 1 proporciona abstracciones que ayudan a los desarrolladores a trabajar con estas limitaciones, pero no simula completamente la mutabilidad. Por lo tanto, los desarrolladores deben pensar cuidadosamente en cómo administran la memoria y las estructuras de datos en sus programas para optimizar el rendimiento.

## Referencias

- Arquitectura del CPU de Cairo: <https://eprint.iacr.org/2021/1063>
- Cairo, Sierra y Casm: <https://medium.com/nethermind-eth/under-the-hood-of-cairo-1-0-exploring-sierra-7f32808421f5>
- Estado no determinista:  <https://twitter.com/PapiniShahar/status/1638203716535713798>

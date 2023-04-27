# ABIs e Interfaces de Contrato

Las interacciones entre contratos inteligentes en una cadena de bloques, también conocidas como "cross-contract", son una práctica común que nos permite construir contratos flexibles que puedan comunicarse entre sí.

Para lograr esto en Starknet, se requiere algo que llamamos una interfaz.

## Interfaz
Una interfaz es una lista de definiciones de funciones de un contrato sin implementaciones. En otras palabras, una interfaz especifica las declaraciones de función (nombre, parámetros, visibilidad y valor de retorno) contenidas en un contrato inteligente sin incluir el cuerpo de la función.

Las interfaces en Cairo son rasgos con el atributo `[abi]`. Si eres nuevo en los rasgos, consulta el capítulo dedicado a [rasgos](./ch07-02-traits-in-cairo.md).

Para que tu código de Cairo califique como una interfaz, debe cumplir con los siguientes requisitos:

1. Debe estar marcado con el atributo `[abi]`.
2. Las funciones de tu interfaz no deben tener implementaciones.
3. Debes declarar explícitamente el decorador de la función.
4. Tu interfaz no debe declarar un constructor.
5. Tu interfaz no debe declarar variables de estado.

Aquí hay un ejemplo de una interfaz para un contrato de token ERC20:

```rust
use starknet::ContractAddress;

#[abi]
trait IERC20 {
    #[view]
    fn name() -> felt252;

    #[view]
    fn symbol() -> felt252;

    #[view]
    fn decimals() -> u8;

    #[view]
    fn total_supply() -> u256;

    #[view]
    fn balance_of(account: ContractAddress) -> u256;

    #[view]
    fn allowance(owner: ContractAddress, spender: ContractAddress) -> u256;

    #[external]
    fn transfer(recipient: ContractAddress, amount: u256) -> bool;

    #[external]
    fn transfer_from(sender: ContractAddress, recipient: ContractAddress, amount: u256) -> bool;

    #[external]
    fn approve(spender: ContractAddress, amount: u256) -> bool;
}
```

<span class="caption">Listado 9-1: Una interfaz simple de ERC20</span>

## ABIs
ABI significa Interfaz Binaria de Aplicaciones. Los ABI dan a un contrato inteligente la capacidad de comunicarse e interactuar con aplicaciones externas u otros contratos inteligentes. Los ABI se pueden comparar con las API en el desarrollo web tradicional, que ayudan al flujo de datos entre aplicaciones y servidores.

Si bien escribimos nuestras lógicas de contrato inteligente en Cairo de alto nivel, se almacenan en la VM como bytecodes ejecutables que están en formatos binarios. Dado que este bytecode no es legible por humanos, requiere interpretación para ser entendido. Aquí es donde entran en juego los ABI, definiendo métodos específicos que se pueden llamar a un contrato inteligente para su ejecución.

Cada contrato en Starknet tiene una Interfaz Binaria de Aplicaciones (ABI) que define cómo codificar y decodificar datos al llamar a los métodos del contrato inteligente.

En el próximo capítulo, veremos cómo podemos llamar a otros contratos inteligentes utilizando un `Contract Dispatcher`, un `Library Dispatcher`, y `System calls`.
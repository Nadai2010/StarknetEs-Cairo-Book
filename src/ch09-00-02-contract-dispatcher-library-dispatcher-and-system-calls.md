# Despachador de Contratos, Despachador de Bibliotecas y Llamadas del Sistema

Cada vez que se crea una interfaz de contrato en Starknet, se crean automáticamente y exportan dos despachadores:
1. El Despachador de Contratos
2. El Despachador de Bibliotecas

En este capítulo, discutiremos en detalle cómo funcionan estos despachadores y su uso.

Para desglosar efectivamente los conceptos en este capítulo, utilizaremos la interfaz IERC20 del capítulo anterior (consulte la Lista 9-1):

## Despachador de Contratos
Los contratos anotados con el atributo `abi` están programados para generar automáticamente y exportar la lógica de despachador relevante durante la compilación. El compilador también genera un nuevo trait, dos nuevas estructuras (una para llamadas de contrato y otra para llamadas de biblioteca) y su implementación de este trait. Nuestra interfaz se expande en algo como esto:

```rust
trait IERC20DispatcherTrait<T> {
    fn get_name(self: T) -> felt252;
    fn transfer(self: T, recipient: ContractAddress, amount: u256);
}

#[derive(Copy, Drop)]
struct IERC20Dispatcher {
    contract_address: starknet::ContractAddress,
}

impl IERC20DispatcherImpl of IERC20DispatcherTrait::<IERC20Dispatcher> {
    fn get_name(self: IERC20Dispatcher) -> felt252 {
        // starknet::call_contract_syscall is called in here
    }
    fn transfer(self: IERC20Dispatcher, recipient: ContractAddress, amount: u256) {
        // starknet::call_contract_syscall is called in here
    }
}
```
<span class="caption">Listado 9-2: Una forma expandida de la interfaz IERC20</span>

**Nota:** El código expandido para nuestra interfaz IERC20 es mucho más robusto, pero para mantener este capítulo conciso y al grano, nos enfocamos en una función de vista `get_name` y una función externa `transfer`.

También es digno de mención que todo esto se abstrae detrás de escena, gracias al poder de los complementos de Cairo.

### Llamando contratos usando el Dispatcher de Contrato
Llamar a otro contrato, digamos `ContractA` usando el dispatcher de interfaz de contrato, llama la lógica de `ContractA` en su contexto, y en la mayoría de los casos puede alterar el estado de `ContractA`. Aquí hay un ejemplo:

```rust
//**** Specify interface here ****//

#[contract]
mod Dispatcher {
    use super::IERC20DispatcherTrait;
    use super::IERC20Dispatcher;
    use starknet::ContractAddress;

    #[view]
    fn token_name(
        _contract_address: ContractAddress
    ) -> felt252 {
        IERC20Dispatcher {contract_address: _contract_address }.name()
    } 

    #[external]
    fn transfer_token(
        _contract_address: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool {
        IERC20Dispatcher {contract_address: _contract_address }.transfer(recipient, amount)
    } 
}
```
<span class="caption">Listado 9-3: Un contrato de muestra que utiliza el Dispatcher de Contratos</span>

Como se puede observar, primero tuvimos que importar `IERC20DispatcherTrait` e `IERC20Dispatcher`, los cuales fueron generados y exportados al compilar nuestra interfaz. Luego realizamos llamadas a los métodos implementados para la estructura `IERC20Dispatcher` (`name`, `transfer`, etc.), pasando el parámetro `contract_address` que representa la dirección del contrato que queremos llamar.

## Dispatcher de Biblioteca
La principal diferencia entre el Dispatcher de Contratos y el Dispatcher de Biblioteca es que, mientras que el Dispatcher de Contratos llama a la lógica de un contrato externo en el contexto del contrato externo, el Dispatcher de Biblioteca llama al hash de clase del contrato objetivo mientras ejecuta la llamada en el contexto del contrato que llama. 
Por lo tanto, a diferencia del Dispatcher de Contratos, las llamadas realizadas utilizando el Dispatcher de Biblioteca no tienen la posibilidad de manipular el estado del contrato objetivo.

Como se indicó en el capítulo anterior, los contratos anotados con la macro `#[abi]` en la compilación generan un nuevo trait, dos nuevas estructuras (una para llamadas a contratos y otra para llamadas a bibliotecas) y su implementación de este trait. La forma expandida de los traits de biblioteca se ve así: 

```rust
trait IERC20DispatcherTrait<T> {
    fn get_name(self: T) -> felt252;
    fn transfer(self: T, recipient: ContractAddress, amount: u256);
}

#[derive(Copy, Drop)]
struct IERC20LibraryDispatcher {
    class_hash: starknet::ClassHash,
}

impl IERC20LibraryDispatcherImpl of IERC20DispatcherTrait::<IERC20LibraryDispatcher> {
    fn get_name(self: IERC20LibraryDispatcher) -> felt252 {
        // starknet::syscalls::library_call_syscall  is called in here
    }
    fn transfer(self: IERC20LibraryDispatcher, recipient: ContractAddress, amount: u256) {
        // starknet::syscalls::library_call_syscall  is called in here
    }
}
```
<span class="caption">Listado 9-4: Una forma expandida del trait IERC20</span>

### Llamando a Contratos usando el Dispatcher de Biblioteca
A continuación se muestra un código de muestra sobre cómo llamar a contratos utilizando el Dispatcher de Biblioteca:

```rust
//**** Specify interface here ****//

use super::IERC20DispatcherTrait;
use super::IERC20LibraryDispatcher;
use starknet::ContractAddress;

#[view]
fn token_name() -> felt252 {
    IERC20LibraryDispatcher { class_hash: starknet::class_hash_const::<0x1234>() }.name()
} 

#[external]
fn transfer_token(
    recipient: ContractAddress, amount: u256
) -> bool {
    IERC20LibraryDispatcher { class_hash: starknet::class_hash_const::<0x1234>() }.transfer(recipient, amount)
} 
```
<span class="caption">Listado 9-4: Un contrato de muestra que utiliza el Dispatcher de Biblioteca</span>

Como se puede ver, primero tuvimos que importar `IERC20DispatcherTrait` e `IERC20LibraryDispatcher`, los cuales fueron generados y exportados al compilar nuestra interfaz. Luego realizamos llamadas a los métodos implementados para la estructura `IERC20LibraryDispatcher` (`name`, `transfer`, etc.), pasando el parámetro `class_hash` que representa la clase del contrato que queremos llamar.

## Llamando a Contratos usando llamadas de sistema de bajo nivel
Otra forma de llamar a otros contratos es mediante la llamada de sistema `starknet::call_contract_syscall`. Los Dispatchers que describimos en las secciones anteriores son sintaxis de alto nivel para esta llamada de sistema de bajo nivel.

El uso de la llamada de sistema `starknet::call_contract_syscall` puede ser útil para la personalización del manejo de errores o para tener más control sobre la serialización/deserialización de los datos de llamada y los datos devueltos. Aquí hay un ejemplo que demuestra una llamada de `transfer` de bajo nivel: 

```rust
#[external]
fn transfer_token(
    address: starknet::ContractAddress, selector: felt252, calldata: Array<felt252>
) -> Span::<felt252> {
    starknet::call_contract_syscall(
        :address, entry_point_selector: selector, calldata: calldata.span()
    ).unwrap_syscall()
} 
```

<span class="caption">Listado 9-5: Un contrato de muestra que implementa llamadas de sistema</span>

Como se puede ver, en lugar de pasar nuestros argumentos de función directamente, pasamos la dirección del contrato, el selector de la función (que es un hash keccak del nombre de la función) y los datos de llamada (argumentos de función). Al final, se nos devuelve un valor serializado que tendremos que deserializar nosotros mismos.
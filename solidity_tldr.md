
## solidity tl;dr

<br>

### ethereum contracts

<br>

* until [account abstract](https://github.com/go-outside-labs/mev-toolkit/tree/main/MEV_by_chains/MEV_on_Ethereum/account_abstraction) becomes a thing, there are two types of accounts, which are identified by an address of **160-bit length** (rightmost 160 bits of the **Keccak hash** of the RLP encoding of the structure with the sender and the nonce), and contain a **balance**, a **nonce**, a **bytecode**, and **stored data** (storage). while **external accounts have a private key** and their code and storage are empty, **contract accounts store their bytecode** (and merkle root hash of the entire state tree).

<br>

* the **creation of a contract** is a transaction where the **receiver address is empty** and its **data field contains compiled bytecode** (or calling `CREATE` opcode. the data sent is executed as bytecode, initializing the state variables in storage and determining the body of the contract being created.

<br>

* contracts can call contracts through **message calls** (`CALL` opcode). every call has a **sender**, a **recipient**, a **payload** (data), a **value** (in wei), and some **gas**. a variant is `DELEGATECALL`, where target code is executed within the context of the calling contract, and `msg.sender` and `msg.value` do not change (the contract can dynamically load code - storage - from a different address at runtime - while current address and balance still refer to the calling contract).

<br>

------

### predefined global variables and functions

<br>

* when a contract is executed in the EVM, it has access to a small set of global objects: `block`, `msg`, and `tx` objects. in addition, solidity exposes a number of EVM opcodes as predefined functions:


<br>

##### msg

* `msg object`: the transaction that triggered the execution of the contract.
* `msg.sender`: sender address of the transaction.
* `msg.value`: ether sent with this call (in wei).
* `msg.data`: data payload of this call into our contract.
* `msg.sig`: first four bytes of the data payload, which is the function selector.

<br>

##### tx

* `tx.gasprice`: gas price in the calling transaction.
* `tx.origin`: address of the originating EOA for this transaction. WARNING: unsafe!

<br>

##### block

* `block.coinbase`: address of the recipient of the current block's fees and block reward.
* `block.gaslimit`: maximum amount of gas that can be spent across all transactions included in the current block.
* `block.number`: current block number (blockchain height).
* `block.timestamp`: timestamp placed in the current block by the miner (number of seconds since the Unix epoch).

<br>

##### address

* `address.balance`: balance of the address, in wei. 
* `address.transfer(__amount__)`: transfers the amount (in wei) to this address, throwing an exception on any error.
* `address.send(__amount__)`: similar to transfer, only instead of throwing an exception, it returns false on error. WARNING: always check the return value of send.
* `address.call(__payload__)`: low-level `CALL` function—can construct an arbitrary message call with a data payload. Returns false on error. WARNING: unsafe.
* `address.delegatecall(__payload__)`: low-level `DELEGATECALL` function, like `callcode(...)` but with the full msg context seen by the current contract. Returns false on error. WARNING: advanced use only!


<br>

##### built-in functions

* `addmod`, `mulmod`: for modulo addition and multiplication. for example, `addmod(x,y,k)` calculates `(x + y) % k`.
* `keccak256`, `sha256`, `sha3`, `ripemd160`: calculate hashes with various standard hash algorithms.
* `ecrecover`: recovers the address used to sign a message from the signature.
* `selfdestruct(__recipient_address__)`: deletes the current contract, sending any remaining ether in the account to the recipient address.
* `this`: address of the currently executing contract account.

<br>

##### what is considered modifying state

- writing to state variables
- emitting events
- creating other contracts
- sending ether via calls
- using selfdestruct
- using low-level calls
- calling any function not marked view or pure
- using inline assembly that contains certain opcodes





<br>

---

### solidity vs. python/js/c++


<br>

from python, we get: 
- modifiers
- multiple inheritances

from js we get:
- function-level scoping
- the `var` keyword

from c/c++ we get:

- scoping: variables are visible from the point right after their declaration until the end of the smallest {}-block that contains the declaration.
- the good ol' value types (passed by value, so they are alway copied to the stack) and reference types (references to the same underlying variable).
- however, look how cool: a variable that is declared will have an initial default value whose byte-representation is all zeros.
- int and uint integers, with uint8 to uint256 in step of 8.

from being statically-typed:
- the type of each variable (local and state) needs to be specified at compile-time (as opposed to runtime).

<br>

you start files with the `SPDX License Identifier (`// SPDX-License-Identifier: MIT`)`. SPDX stands for software package data exchange. The compiler will include this in the bytecode metadata and make it machine readable.

<br>

**pragmas:** directives that are used to enable certain compiler features and checks. 

version Pragma indicates the specific Solidity compiler version. It does not change the version of the compiler, though, so yeah, you will get an error if it does not match the compiler.

other types are Compiler version, ABI coder version, SMTCheker.

<br>

The best-practices for layout in a contract are:
1. state variables
2. events
3. modifiers
4. constructors
5. functions

<br>


**natspec comments**: Also known as the "ethereum natural language specification format". Written as triple slashes (`///`) or double asterisk block
`(/**...*/)`, directly above function declarations or statements to generate documentation in `JSON` format for developers and end-users. These are some tags:

* `@title`: describe the contract/interface
* `@author`
* `@notice`: explain to an end user what it does
* `@dev`: explain to a dev 
* `@param`: document params
* `@return`: any returned variable
* `@inheritdoc`: copies missing tags from the base function (must be followed by contract name)
* `@custom`: anything application-defined



<br>

**events**: an abstraction on top of EVM's logging: emitting events cause the arguments to be stored in the transaction's log (which are associated with the address of the contract). events are emitted using **emit**.

events are especially useful for light clients and DApp services, which can "watch" for specific events and report them to the user interface, or make a change in the state of the application to reflect an event in an underlying contract.

<br>


---

### type of variables

<br>

**address types**. the address type comes in two types:

1. holds a 20 byte value (the size of an Ethereum address)
2. address payable: with additional members transfer and send. address payable is an address you can send Ether to (while plain address not).

explicit conversion from address to address payable can be done with `payable()`.
explicit conversion from or to address is allowed for `uint160`, integer literals, `byte20`, and contract types

the members of address type are pretty interesting: `.balance`, `.code`, `.codehash`, `.transfer`, `.send`, `.call`, `.delegatecall`, `.staticcall`.

<br>

**fixed-size Byte Arrays**: bytes1, bytes2, bytes3, …, bytes32 hold a sequence of bytes from one to up to 32. The type `byte[]` is an array of bytes, but due to padding rules, it wastes 31 bytes of space for each element, so it's better to use `bytes()`


<br>

**state variables**: variables that can be accessed by all functions of the contract and values are permanently stored in the contract storage.

**state visibility specifiers**: these are state variables that define how the methods will be accessed:

- `public`: part of the contract interface and can be accessed internally or via messages.
- `external`: like public functions, but cannot be called within the contract.
- `internal`: can only be accessed internally from within the current contracts (or contracts deriving from it).
- `private`: can only be accessed from the contract they are defined in and not in derived contracts.
- `pure`: neither reads nor writes any variables in storage. It can only operate on arguments and return data, without reference to any stored data. Pure functions are intended to encourage declarative-style programming without side effects or state.
- `payable`: can accept incoming payments. Functions not declared as payable will reject incoming payments. There are two exceptions, due to design decisions in the EVM: coinbase payments and `SELFDESTRUCT` inheritance will be paid even if the fallback function is not declared as payable.



**immutability**: state variables can be declared as constant or immutable, so they cannot be modified after the contract has been constructed. their difference is beautiful:

**for constant variables, the value is fixed at compile-time; for immutable variables, the value can still be assigned at construction time (in the constructor or point of declation)**

there is an entire gas cost thing too. For constant variables, the expression assigned is copied to all the places, and re-evaluated each time (local optimizations are possible). For immutable variables, the expression is evaluated once at constriction time and their value is copied to all the places in the code they are accessed, on a reserved 32 bytes, becoming usually more expensive than constant.

<br>

---

### functions

<br>

**functions modifiers**: used to change the behavior of functions in a declarative way, so that the function's control flow continues after the "_" in the preceding modifier. This symbol can appear in the modifier multiple times. 

the underscore followed by a semicolon is a placeholder that is replaced by the code of the function that is being modified. Essentially, the modifier is "wrapped around" the modified function, placing its code in the location identified by the underscore character.

to apply a modifier, you add its name to the function declaration. More than one modifier can be applied to a function; they are applied in the sequence they are declared, as a space-separated list.

```
function destroy() public onlyOwner {
```

<br>

**function visibility specifiers**: these are how visibility works for functions:

- `public`: part of the contract interface and can be either called internally or via messages. 
- `external`: part of the contract interface, and can be called from other contracts and via transactions. Here is the interesting part: an external function `func` cannot be called internally, so `func()` would not work. But `this.func()` does.
- `internal`: can only be accessed from within the current contract or contracts deriving from it.
- `private`: can only be accessed from the contract they are defined in and not even in derived contracts

<br>

**function mutability specifiers**:

- `view` functions can read the contract state but not modify it: enforced at runtime via STATICALL opcode.
- `pure` functions can neither read a contract nor modify it.
- only view can be enforced at the EVM level, not pure.

<br>

**overloading**: a contract can have multiple functions of the same name but with different parameter types. they are matched by the arguments supplied in the function call 😬.


<br>

---

### data structures

<br>

- `structs`: custom-defined types that can group several variables of same/different types together to create a custom data structure.
- `enums`: used to create custom types with a finite set of constants values. Cannot have more than 256 members.

<br>

**constructors**: when a contract is created, the function with *constructor* is executed once and then the final code of the contract is stored on the blockchain (all public and external functions, but not the constructor code or internal functions called by it).

<br>

**receive function**: a contract can have ONE *receive* function (*receive() external payable {...}*) without the function keyword, and no arguments and no return and... have `external` and `payable`. this is the function on plain ether transfers via `send()` or `transfer()`.

interesting facts:

- receive is executed on a call to the contract with empty calldata.
- receive might only rely on 2300 gas being available.
- a contract without Receive can actually receive Ether as a recipient of a coinbase transaction (miner block reward) or as a destination of `selfdestruct`.
- a contract cannot react to the Ether transfer above.

<br>

**falback function**: kinda in the same idea, a contract can have ONE *fallback* function, which must have external visibility.

- fallback is executed on a call to the contract if none of the other functions match the given function signature or no data was supplied and there is not receive Ether function.


<br>

**transfer:** the transfer function fails if the balance of the contract is not enough or if the transfer is rejected by the receiving account, revering on failure.

<br>

**send:** low-level counterpart of transfer, however, if the execution fails then send only returns false (return value must be checked by the caller).

<br>

----

### data management

<br>

* the evm manages different kinds of data depending on their context:

<br>

* **stack**: the evm operates on a virtual stack, which has a maximum size of 1024, stack items have a size of 256 bits (the evm is a 256-bit word machine, which facilitates keccak256 hash scheme and elliptic-curve). the opcodes to modify the stack are: `POP` (remove from stack), `PUSH n` (places the `n` butes item into the stack), `DUP n` (duplicates the `n`th stack item), `SWAP n` (exchanges the first and the `n`th stack item).

* **calldata**: read-only byte-addressable space where the data parameter of a tx or call is held. unlike the stack, to use this data, you have to specify an exact byte offset and number of bytes to read. the opcodes include: `CALLDATASIZE` (get size of tx data), `CALLDATALOAD` (loads 32 byte of tx data onto the stack), `CALLDATACOPY` (copies the number of bytes of the tx data to memory). there are also the inline assembly versions: `calldatasize`, `calldataload`, calldatacopy`. they can be called through:

```
assembly {
}
```

* **memory**: volatile read-write byte-addressable space (store data during execution) initiallized as zero. the evm opcodes are `MLOAD` (loads a word into the stack), `MSTORE` (saves a word to memory), `MSTORE8` (saves a byte to memory).

* **storage**: persistant read-write word-addressable space for contracts, addressed by words. it's a key-value mapping of 2**256 slots of 32 bytes each. gas to save data into storage is one of the highest operations. the evm opcodes are: `SLOAD` (loads a word from storage to stack), `SSTORE` (saves a word to storage).


<br>

----

### calling another contract

<br>

**call/delegatecall/ataticall**: ued to interface with contracts that do not adhere to ABI, or to give more direct control over encoding. they all take a single bytes memory parameter and return the success condition (as a bool) and the return data (byte memory).

with `DELEGATECALL`, only the code of the given address is used but all other aspects are taken from the current contract. The purpose is to use logic code that is stored in the callee contract but operates on the state of the caller contract.

with `STATCALL`, the execution will revert if the called function modifies the state in any way.

<br>


**creating a new instance**:

* the safest way to call another contract is if you create that other contract yourself. 
* to do this, you can simply instantiate it, using the keyword `new`, as in other object-oriented languages. This keyword will create the contract on the blockchain and return an object that you can use to reference it. 

```
contract Token is Mortal {
	Faucet _faucet;

    constructor() {
        _faucet = new Faucet();
    }
}
```

<br>

**addressing an existing instance**:

* another way you can call a contract is by casting the address of an existing instance of the contract. 
* with this method, you apply a known interface to an existing instance.
* this is much riskier than the previous mechanism, because we don’t know for sure whether that address actually is a faucet object.

```
import "Faucet.sol";

contract Token is Mortal {

    Faucet _faucet;

    constructor(address _f) {
        _faucet = Faucet(_f);
        _faucet.withdraw(0.1 ether);
    }
}

```



<br>

----

### block and tx properties** 

- `blockhash`
- `block.chainid`
- `block.coinbase`
- `block.difficulty`
- `block.gaslimit`
- `block.number`
- `block.timestamp`
- `msg.data`
- `msg.sender`
- `msg.sig`
- `msg.value`
- `tx.gasprice`
- `gasleft`
- `tx.origin`

<br>

**randomness**. Not cute shit: you cannot rely on block.timestamp or blockhash as a source of randomness, as they can be influenced by miners to some degree.

<br>

---

### ABI encoding and decoding functions

<br>

- `abi.decode`
- `abi.encode`
- `abi.encodePacked`
- `abi.encodeWithSelector`
- `abi.encodeWithSignature`

<br>

----

### error handling

<br>

- `assert()`: causes a panic error and reverts if the condition is not met
- `require()`: reverts if the condition is not met
- `revert()`: abort execution and revert state changes


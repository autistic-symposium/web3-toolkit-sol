## tricks to save gas


### tl; dr

- gas is the cost for on-chain computation and storage.
- examples: addition costs 3 gas, Keccak-256 costs 30 gas + 6 gas for each 256 bits of data being hashed, and sending a transaction costs 21,000 gas.
- brute force hashes of function names to find those that start 0000, so this can save around 50 gas.
- avoid calls to other contracts.
- ++i uses 5 gas less than i++.
- if you don’t need a variable anymore, you should delete it using the delete keyword provided by solidity or by setting it to its default value.
<br>

#### pack variables

The below code is an example of poor code and will consume 3 storage slot:

```
uint8 numberOne;
uint256 bigNumber;
uint8 numberTwo;
```

A much more efficient way to do this in solidity will be:

```
uint8 numberOne;
uint8 numberTwo;
uint256 bigNumber;
```



#### constant vs. immutable variables

Constant values can sometimes be cheaper than immutable values:

- For a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time, allowing local optimizations.
- Immutable variables are evaluated once at construction time and their value is copied to all the places in the code where they are accessed. For these values, 32 bytes are reserved, even if they would fit in fewer bytes. 


#### mappings are cheaper than Arrays

- avoid dynamically sized arrays
- An array is not stored sequentially in memory but as a mapping.
- You can pack Arrays but not Mappings.
- It’s cheaper to use arrays if you are using smaller elements like `uint8` which can be packed together.
- You can’t get the length of a mapping or parse through all its elements, so depending on your use case, you might be forced to use an Array even though it might cost you more gas.


#### use bytes32 rather than string/bytes

- If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity.
- Any fixed size variable in solidity is cheaper than variable size.

#### modifiers

- For all the public functions, the input parameters are copied to memory automatically, and it costs gas.
- If your function is only called externally, then you should explicitly mark it as external.
- External function’s parameters are not copied into memory but are read from `calldata` directly.
- internal and private are both cheaper than public and external when called from inside the contract in some cases.



#### no need to initialize variables with default values

- If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you are just wasting gas.

```
uint256 hello = 0; //bad, expensive
uint256 world; //good, cheap
```


#### make use of single line swaps 

- This is space-efficient:

```
(hello, world) = (world, hello)
```

#### negative gas costs

- Deleting a contract (SELFDESTRUCT) is worth a refund of 24,000 gas.
- Changing a storage address from a nonzero value to zero (SSTORE[x] = 0) is worth a refund of 15,000 gas.

<br>


---

### resources 



* [truffle contract size](https://github.com/IoBuilders/truffle-contract-size)
* [solidity gas optimizations](https://mirror.xyz/haruxe.eth/DW5verFv8KsYOBC0SxqWORYry17kPdeS94JqOVkgxAA)
* [hardhat](https://medium.com/@thelasthash/%EF%B8%8F-gas-optimization-with-hardhat-1e553eaea311)
* [foundry](https://book.getfoundry.sh/forge/gas-reports)


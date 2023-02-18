## calldata

<br>

### tl; dr

<br>

* calldata is the encoded parameter(s) sent on functions by smart contracts to the evm (for example, through `abi.econde()`,`abi.econcdeWithSelector() for a particular interfaced function, or `abi.encodePacked()` for efficient dynamic variables). 
* `selector()` generates the 4-bytes representing the method in the interface: this is how the evm knows which function is being interacted.

* each piece of calldata is 32 bytes long (64 chars), where 20 hex == 32-bytes
* types:
   - static variables: the encoded representation of `uint`, `address`, `bool`, `bytes`, `tuples`
   - dynamics variables: non-fixed-size types: `bytes`, `string`, dynamic and fixed arrays `<T>[]`

<br>


---

### resources

<br>

* [mev memoirs in the arena, by noxx](https://noxx.substack.com/p/mev-memoirs-into-the-arena-chapter?s=r)

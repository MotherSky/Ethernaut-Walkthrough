# `Fal1out`

To hack this level, we will need to claim the ownership of the smart contract.

## Code Breakdown

[The Smart Contract's Code can be seen here.](Fallback.sol)

The solution to this level is quite easy. If we read the whole contract code, it seems like a normal contract with no way to get ownership.
The trick resides in the constructor:

```solidity
/* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }
```
A constructor, much like other programming languages is a function that is only  executed once upon contract creation. It is generally used for initializing variables and setting ownership.
the function name here is misspelled, it is called `Fal1out` instead of `Fallout` wich means anyone can call this function and get ownership.
```js
>  await contract.fal1out()
```
`Note: This code is deprecated. In versions prior to 0.4.22, constructors were defined as functions with the same name as the contract. This syntax was deprecated (After the Rubixi hack) and is not allowed anymore in version 0.5.0 and above. Thus the contract's code will not compile `
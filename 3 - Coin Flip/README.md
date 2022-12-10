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
This contract only consists of one main function `flip()`. It determines the outcome of a "random" coinflip using a hash of the latest `block number` using the following formula: 
```solidity
uint256(blockhash(block.number - 1)) / FACTOR;
```
This will generate either 1 or 0.
The main problem with this method of generating "randomness" is that at first it seems that the coinflip is indeed random but in fact, everyone can get see the block number and malicious miners can even influence it by modify block headers or choose not to publish blocks in order to influence the resulting hashes
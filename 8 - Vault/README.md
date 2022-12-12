# Vault

Unlock the vault to pass the level!

## Code Breakdown

[The Smart Contract's Code can be seen here.](Vault.sol)

```solidity
    bool public locked;
    bytes32 private password;
```

This contract contains 2  state variables:
* locked: a boolean, this is set to true at deployment.
* password: a ***private*** bytes32 that is also set at deployment:
```solidity
constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }
```

And one main function that unlocks the vault if given the correct password:
```solidity
function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
```

## Level Completion

Everything used in a smart contract is publicly visible, even local variables and state variables marked ***private***.
Marking a variable private only only prevents other contracts from accessing it.
We can access to the password value from the storage using [web3.eth.getStorageAt](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#getstorageat):

```js
>  (await web3.eth.getStorageAt(contract.address, 1)).toString() // read store at index 1
<· "0x412076657279207374726f6e67207365637265742070617373776f7264203a29" // this is the password value
```
Let's unlock the vault now:
```js
>  await contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
```

**`To ensure that data is private, it needs to be encrypted before being put onto the blockchain. The decryption key should never be sent on-chain. [zk-SNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell) provide a way to determine whether someone possesses a secret parameter, without ever having to reveal the parameter.`**
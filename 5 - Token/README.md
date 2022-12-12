# Token

The goal of this level is to hack this basic token contract, we will be given 20 tokens initially and we should get hold of more.

## Code Breakdown

[The Smart Contract's Code can be seen here.](Token.sol)

```solidity
    mapping(address => uint) balances;
    uint public totalSupply;
```

This contract contains two public variables:
* balances: a mapping to store the address of each holder of the token and its **uint256** balance.
* totalSupply: total supply of the token which will be specified at deployment.

```solidity
constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }
```
The constructor sets the deployer's balance to all of the total supply.

```solidity
function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }
```
The `transfer` function takes an address to send the tokens, and value of token to be sent.
First it requires that the balance of the sender minus the amount must be positive.
This is a poor way of checking whether the sender has enough tokens to send since all the balances and the amount are both **uint** so their subtraction can't be a negative number so it will go through anyway.

the next lines subtracts the amount of tokens sent from the sender's balance then adds it to the recipient's.

```solidity
function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
```
`balanceOf` takes an address and returns its balance.

## Level Completion

if we try to send more tokens ***(21)*** than we already have ***(20)*** .The following line the require will try subtracting 21 from our balance (20), and since the balances are ***uint*** this will lead to an underflow, meaning our new balance will be equal to ***2<sup>256</sup>-1***.

```js
>  await contract.transfer("0x000000000000000000000000000000000000dead", 21)
```
Now let's check our balance:
```js
> (await contract.balanceOf("0xa0D3DFEE9945d53532aDaF60acB4421de5baecA8")).toString()
<· "115792089237316195423570985008687907853269984665640564039457584007913129639935"
```

***Note: SafeMath is a famous library that automatically checks for overflows/underflows in all the mathematical operators to prevent them.***
***Since Solidity 0.8, the overflow/underflow check is implemented on the language level - it adds the validation to the bytecode during compilation.***
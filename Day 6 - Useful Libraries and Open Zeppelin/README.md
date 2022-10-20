Introduction:
- There are various advantages to using a smart contract library for your project. To begin with, it saves you time by offering ready-to-use building pieces that you can include in your system rather than having to write them yourself. 
- A big advantage is security. Smart contract libraries that are open source are often investigated. 
- Because so many projects rely on them, the community has a strong motivation to maintain them under continual scrutiny. 
- Errors in application code are far more prevalent than in reusable contract libraries. External audits are also performed on some libraries for added security.

- Libraries that are well-tested, reusable, and secure become increasingly crucial. In this article, we are going to create an ERC-20 token with a custom supply using Open Zeppelin. 
- Continue reading to find out more about prevalent Ethereum libraries and how they enable efficient contract creation.

SafeMath:
- Solidity's arithmetic operations wrap on overflow. Because programmers commonly think that an overflow causes an error, which is the conventional behavior in high-level programming languages, this can easily lead to problems. 
- When an operation overflows, SafeMath restores this understanding by reversing the transaction.Using this library instead of unchecked operations avoids a whole class of problems, hence it's always advised to use it.

DS Math:
- For the most common numerical primitive types in Solidity, DS-Math provides arithmetic functions. 
- Without the danger of integer overflow, you can securely add, subtract, multiply, and divide uint values. 
- The minimum and maximum of two numbers can also be found.
- This package also contains arithmetic functions for two new higher-level numerical concepts: wad (18 decimals) and ray (27 decimals). 
- Fixed-point decimal numbers are represented by these. 
- This library (rather than basic contracts) offer overflow checks for arithmetic functions not given by the language. 
- It's a good idea to use this library instead of native arithmetic operations to protect your contract from overflows, which can be fatal!

Open Zeppelin:
- OpenZeppelin is a library for developing safe smart contracts created by the Zeppelin team. 
- It includes ERC20 and ERC721 implementations (which you can use as-is or modify to fit your needs), as well as Solidity components for creating bespoke contracts and more complicated decentralized systems. 
- It is hailed as the golden standard for constructing secure blockchain applications. 
- OpenZeppelin makes security technologies for developing, automating, and running decentralized applications. 
- They also safeguard top companies by conducting security assessments on their systems and goods. 
- It provides you with a library of modular, secure, and efficient smart contracts for the Ethereum system written in Solidity. 
- Open Zeppelin is not only the most popular library in the industry but it also reduces your attack surface by reusing audited code. 
- Now let's move on to creating an ERC-20 token with a custom supply.

Using Open Zeppelin Libraries:
- We will now explore using multiple Open Zeppelin libraries for some fruitful use cases in your smart contracts. 
- As a rule of thumb, you need to import the respective libraries into your contract by using the import statement before declaring your contract. 
- We will import ERC20.sol for reference purposes only (on the Remix IDE). Note that for inheriting a library, the term is is used:

```
//SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.9;
import "github/OpenZeppelin/openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
contract myERC20 is ERC20 {}
```

Constructing ERC-20 Token with Custom Supply Mechanism:
- Now we will guide you on how to create an ERC-20 token with a custom supply mechanism. 
- This tutorial will showcase two ways to use OpenZeppelin contracts for the said purpose. 
- It is not practical to deploy ERC-20 without any supply, therefore we need to define the way in which that supply is created.

Fixing Supply:
- Let's say you want a token with the fixed supply of 5000, initially allocated to the account that deploys the contract. 
- You will input the following code in case of Contracts v1:

```
contract ERC20FixedSupply is ERC20 {
	constructor() public {
		totalSupply += 5000;
		balances[msg.sender] += 5000;
	}
}
```

- This approach is not only discouraged but also forbidden in Contracts v2. 
- TotalSupply and balances are now secret ERC20 implementation details, and you can't write to them directly. 
- Instead, there is an internal _mint function that will accomplish the same thing:

```
contract ERC20FixedSupply is ERC20 {
    constructor() public {
        _mint(msg.sender, 5000);
    }
}
```

Contract extensions are safer when the state is encapsulated in this way. Now we move on to the part where we reward miners for their efforts and encourage them.

Incentivizing Miners:
- The internal _mint function is a critical component for writing ERC20 extensions that implement a supply mechanism.
- The approach we'll use is a token incentive system for miners that generate Ethereum blocks. 
- The address of the current block's miner may be found in the global variable block. coinbase in Solidity. 
- When someone runs the mintMinerReward() method on our coin, we'll send a token reward to this address:

```
contract ERC20WithMinerReward is ERC20 {
    function mintMinerReward() public {
        _mint(block.coinbase, 5000);
    }
}
```

Modularizing the Mechanism:
- ERC20Mintable is one of the supply mechanisms already incorporated in Contracts. 
- This is a general technique in which the minter role is allocated to a group of accounts, allowing them to call the mint function, which is an external version of _mint.
- This can be used for centralized minting, in which an externally held account (i.e. someone with a pair of cryptographic keys) selects how much supply to produce and to whom it should be distributed. 
- Traditional asset-backed stablecoins, for example, are a perfectly reasonable use case for this technique.The accounts with the minter role, on the other hand, do not have to be externally owned and can be smart contracts that implement a trustless process. 
- The same behavior as in the preceding section can be implemented.

```
contract MinerRewardMinter {
    ERC20Mintable _token;

    constructor(ERC20Mintable token) public {
        _token = token;
    }

    function mintMinerReward() public {
        _token.mint(block.coinbase, 5000);
    }
}
```

- Due to this by assigning the role to different contracts, we can simply combine several supply systems, and we can do it dynamically.

Deploying System for Automatic Rewards:
- Other internal ERC20 operations, such as _transfer, can be utilized or enhanced in addition to _mint. 
- Because this method facilitates token transfers and is utilized by ERC20, it may be used to automatically activate functionality. 
- This is something that the ERC20Mintable method does not allow for.
- We can utilize this to mint a miner reward for every token transfer that is included on the blockchain, in addition to our prior supply system:

```
contract ERC20WithAutoMinerReward is ERC20 {
    function _mintMinerReward() internal {
        _mint(block.coinbase, 5000);
    }

    function _transfer(address from, address to, uint256 value) internal {
        _mintMinerReward();
        super._transfer(from, to, value);
    }
}
```

- Did you notice how we override _transfer to first mint the miner reward before calling super. transfer to perform the old version. 
- This final step is critical for maintaining the original semantics of ERC20 transfers.

Final Thoughts:
- In this tutorial, we saw how one can implement the ERC-20 supply mechanism internally through _mint, and externally through ERC20Mintable. 
- We hope that this article has helped you in understanding the fundamentals of OpenZeppelin and will be crucial for building your own smart contract. Happy coding!

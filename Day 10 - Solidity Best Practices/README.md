Introduction:
- A code that is poorly designed and written is guaranteed to haunt you in the form of technical debt in the longer run. Do you want your code to scare you or your fellow developers in the future? I reckon not!
- Technical debt is something your product or application develops over time when more and more inconsistent and bad development practices become part of your application engineering cycle. 
- The industry firmly believes that once a technical debt is developed in your code, there are lesser chances of turning back. Moreover, it can also drastically contribute to insecure execution. 
- Hence, it is advised that you start following the best coding practices right from the scratch and build a beautiful codebase for the future.
- Welcome to the last day of the series. In this tutorial, we'll discuss some of the best practices that you should follow while programming your contracts in Solidity. 
- Without further ado, let's get it rolling!

Data Privacy:
- Data privacy is the topmost concern these days; especially with all these privacy regulations (GDPR, CCPA, LGPD, PIPL, etc.) in place. 
- Imagine that some malicious activity becomes successful in stealing data from your contracts or adding PII (Personally Identifiable Information) to the Blockchain itself. That's indeed problematic.
- This is where we want to be extra cautious when dealing with personal data; even inside private variables. 
- Generally, we should either try to properly encrypt such data or just simply store it off-chain.


Loop Pagination:
- To execute functions inside a contract, you need a certain amount of gas based on the complexity of computations. 
- The catch here is that Ethereum has a threshold for the amount of gas per block. It essentially means that the sum of gas required for all transactions that happen inside a single block should be well within the Ethereum specified gas limit.
- If your code is not optimized and it is dealing/looping on large arrays, you are most likely to face Denial of Service (DoS) errors; which means that your transactions have exceeded the block gas limit.
- You can fix it by using pagination while looping and distributing your transactions across multiple blocks.

```
contract DummyDatastore {
   uint256[] private _items;
    
    function getItemAtIndex(uint256 index) public view returns (uint256) {
        return _items[index];
    }

    function countItems() public view returns (uint256) {
        return _items.length;
    }

    ...
}
```

```
contract DummyDatastorePaginatedWrapper { 
    function getItemAtIndexPaginated(
        DummyDatastore store, 
        uint256 startIndex, 
        uint256 offset) public view returns(uint256[] memory) { 
        
        uint256[] memory result; 
        require(startIndex + offset 
        <= store.countItems(), "Read index out of bounds"); 

        for (uint256 i = 0; i < offset; ++i) { 
            result[i] = store.getItemAtIndex(i + startIndex); 
        } 
        
        return result; 
    } 
}
```

Method Signature:
- While accepting parameters in contract functions, don't use them blindly. Rather, check for their validity using require statement, wherever applicable.

Third-Party Calls:
- Not all contracts are trusted or secure. As a developer, it is your responsibility to at least make your contract safe and non-vulnerable to maximum threats. 
- The negligence often happens while using third-party calls where we tend to ignore the validation and try to use the returned result response directly.
- For instance, to ensure a reliable experience in your contract, you should always check/validate all the values returned by an external call before using further in your contract.Always check for errors and exceptions. 
- If not handled properly, it can cause your contract to be in a state of DoS.

```
(bool success, ) = dummyAddress.call.value(55)("");
if(!success) {
    // Add your logic here
}
```

Events:
- In your contracts, you should actively use events to record data related change of state. It helps you monitor and track state changes in a contract.


Safe Casting:
- While casting to smaller-sized data types, it is often noticed that casted data becomes unreliable. 
- By unreliable, it means that loss of information might occur during the cast operation. It is recommended to use libraries that are known to handle such edge cases properly. 
- For Solidity, Open Zeppelin SafeCast library is the way to go.

```
import "@openzeppelin/contracts/utils/SafeCast.sol";

contract DummyContract {
    function safeCastToUint64(uint256 input) public pure returns (uint64) {
        return SafeCast.toUint64(input);
    }
}
```

Checks-Effects-Interactions:
- The Checks-Effects-Interactions is a pattern that ensures expected execution of a contract. It prevents any third-party or external call from getting executed prior to successful updates to the contract state.

```
contract DummyContract {
    …

    function withdrawAmount() public {
        uint amountToWithdraw = balances[msg.sender];
        balances[msg.sender] = 0;

        (bool success, ) = msg.sender.call.value(amountToWithdraw)("");

        // The user's balance is already 0, 
        // so future invocations won't withdraw anything
        require(success);
    }
    …
}
```

Watch Out for Reentrancy Vulnerability:
- You might know about the famous reentrancy vulnerability that was found in Uniswap V1. 
- Cutting long story short, it was primarily because of the before and after transfer hooks in ERC777.
- You can use Open Zeppelin's ReentrancyGuard to secure your contracts against reentrancy vulnerability. 

Say No to tx.origin:
- Solidity has a global variable tx.origin. This variable is responsible to hold the address of the account that sent the transaction. 
- The contracts that depend on this variable for authorization are insecure and can easily be hit by phishing attacks.
- A secure alternative is to use msg.sender. Instead of holding the account address, it stores the address that has called or initiated a function or transaction.

```
contract DummyContract {
    address owner;

    function DummyContract() public {
        owner = msg.sender;
    }

    function sendTo(address receiver, uint amount) public {
        require(msg.sender == owner);
        receiver.transfer(amount);
    }
}
```

Access Control on Public Functions:
- Not every function in your contract is meant to be public. We need to be cautious while writing the functions but even more informed while deciding on correct access modifiers for them. 
- It is recommended that you restrict the access as much as you can, to begin with.


Prefer Interface Types:
- If you want to call any other contract inside your contract, you probably will have to store it as well. 
- Developers usually store the address of the other contract inside theirs. This is a bad practice because, this way, you are not taking advantage of the compile-time checks.
- It is recommended to use the interface type or the contract type itself so that it is also checked during compile time.

```
contract DummyContract {
    OtherContract otherContract;

    function callOtherContract() public view returns (bool){
        bool answer = otherContract.check(69,true);
        return answer;
    }
}
```

```
contract OtherContract {
    function check(uint32 x, bool y) public pure returns (bool r) {
        r = x > 32 || y;
    }
}
```

Get Rid of Dead Code:
- Dead code is nothing but a major cause of confusions. It is just residing in your code, contributing nothing, yet requires maintenance. 
- As of now, the Solidity compiler does not throw an error or a warning for this dead code but, as a developer, you should be watchful about it.
- Furthermore, add unit tests to properly check whether your code is performing as intended. 
- Sometimes, you think that you are writing the correct code but it can act as a dead code as well.
- Suppose, the following line of code is added to one of your contract's methods.

```
msg.sender.call.value(address(this).balance);
```

- Now, it is very easy for the naked eye to actually skip what is wrong above. The above line, essentially, is a dead code because it's just a function reference. 
- The function is not being called in there. The correct code should be:

```
msg.sender.call.value(address(this).balance)("");
```

- Unless you've proper unit tests or some kind of testing mechanism in place, you most likely would've missed it!
- In this piece, we discussed some of the best practices that every Solidity developer should follow in order to write safe and informed code that performs the desired operations efficiently and ensures minimum technical debt over time. 
- The 10 days of Solidity series is now complete. I hope you had an enriching experience. Happy Coding!












































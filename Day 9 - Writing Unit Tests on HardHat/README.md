Introduction:
- In May 2015, Airbus A400M crashed near Seville, a city in Spain. Investigations led to the conclusion that it was software vulnerability that forced the essential engine data to erase; eventually causing engine failure in the middle of the flight. Four out of the six crew members were killed who were onboarded in this Airbus A400M cargo flight.
- We, as developers, are associated with a field where even a bit of negligence can cause serious incidents and human deaths. 
- It is extremely important that the software we develop is thoroughly tested and ensured that it works exactly as expected; no under or over execution.
- Generally, as the business demands scalability in software, it becomes a challenge to ensure that every functionality is working as expected even after a minor change. 
- This is where unit tests come to the rescue. Similarly, in the Blockchain ecosystem as well, these unit tests play a vital role in establishing the integrity of a smart contract.
- Enough talking? Let's get straight into how we can write unit tests for our Solidity contracts in the Hardhat ecosystem.

What is Hardhat?
- For revision purposes, Hardhat is a fast-paced, flexible, and extensible Ethereum development environment that is specially built for professionals. 
- Just like any other well-informed framework, it also comes with a minimum development setup to help developers rapidly build, scale and test the contracts.
- Hardhat comes with a built-in local development chain with a variety of supported operations that provides developers a real-life experience of an actual blockchain and gives them the confidence to rapidly innovate development cycles.
- We covered Hardhat setup yesterday and today we'll be focusing on writing unit tests via Hardhat on our ERC-721 contract from Day 7.

ERC-721 Smart Contract:
- For brushing up, here's our complete ERC-721 contract from Day 7:

```
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract HelloNFT is ERC721("HelloNFT", "HNFT") {
    uint tokenId;
    mapping(address=>tokenMetaData[]) public ownershipRecord;

    struct tokenMetaData {
        uint tokenId;
        uint timeStamp;
        string tokenURI;
    }

    function mintToken(address recipient, string memory url) public {
        _safeMint(recipient, tokenId);
        ownershipRecord[recipient].push(tokenMetaData(tokenId, block.timestamp, url));
        tokenId = tokenId + 1;
    }
}
```

- Writing this contract was just one side of the coin. How we do actually verify that the mintToken function is working as expected? Yes, you guessed it right!
- Let's jump on to the next section and write some unit tests.


Writing Unit Tests for Contract on Hardhat:
- Our HelloNFT contract exposes a function to mint the token against the provided URL. Before moving towards writing the unit tests, let's think of some scenarios that should be tested for the mintToken() function. 
- Following are a few cases just for demonstrating purposes; feel free to think and add more scenarios, as needed.
1) The recipient's address should be valid.
2) The token identifier should be incremented after every successful mint.
3) The minted token metadata should contain the correct provided URL resource (tokenURI in this case) that we provided.
4) The URL resource should be assigned to the recipient address ONLY.

- In a project that is created using Hardhat CLI, there is a separate directory that entails all the unit tests. 
- This directory is named test in the Hardhat boilerplate code structure and it resides at the root of the project.
- Let's create a new JavaScript file in the test folder and call it hello-nft-test.js. We will write tests for some of the example scenarios mentioned above.

![b1](https://user-images.githubusercontent.com/60461421/197587348-2bc54671-641d-4d51-92cf-529cdabfb548.png)

- In hello-nft-test.js, start by importing relevant modules that will help us throughout the testing process.

```
const { expect } = require("chai");
const { ethers } = require("hardhat");
```

- Now, let's test some positive and negative cases for each of the above mentioned scenarios.

1) The recipient's address should be valid: 
- For this scenario, we should ideally check that the mintToken function of our contract behaves as expected in case of both, valid and invalid, addresses.
- Note: Hardhat comes with some demo accounts/addresses for development purposes. Run npx hardhat accounts to see the list of demo accounts.
- In the same hello-nft-test.js file, register the test suite as follows:

```
describe("HelloNFT Mint", function () {});
```

- Once it is registered, add the following two tests that check the validity of the address:

```
it("Should not throw BAD ADDRESS CHECKSUM error if correct recipient address is provided", async function () {
       const HelloNFT = await ethers.getContractFactory("HelloNFT");
       const helloNFT = await HelloNFT.deploy();
       await helloNFT.deployed();
      
       try {
           await helloNFT.mintToken("0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", "https://www.google.com");
          
           // If reached here, it means that code did not throw any error so we can simply pass the test
           expect(true).to.equal(true);
       } catch (err) {
           // If any error is thrown by code, in order for this test to pass,
           // we must check that error is not "bad error checksum"
           expect(
               'bad address checksum (argument="address", value="0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", code=INVALID_ARGUMENT, version=address/5.6.0)'
               ).to.equal(err.message);
           }
       }
   );
```

```
it("Should throw BAD ADDRESS CHECKSUM error if correct recipient address is provided", async function () {
       const HelloNFT = await ethers.getContractFactory("HelloNFT");
       const helloNFT = await HelloNFT.deploy();
       await helloNFT.deployed();
      
       try {
           await helloNFT.mintToken("0x0000a0000a0aa00000aaa0aaa00a0a0a0a0a0000", "https://www.google.com");
          
           // If reached here, it means that code did not throw any error so we can simply pass the test
           expect(true).to.equal(true);
       } catch (err) {
           // If any error is thrown by code, in order for this test to pass,
           // we must check that error is not "bad error checksum"
           expect(
               'bad address checksum (argument="address", value="0x0000a0000a0aa00000aaa0aaa00a0a0a0a0a0000", code=INVALID_ARGUMENT, version=address/5.6.0)'
               ).to.equal(err.message);
           }
       }
   );
```

- Run npx hardhat test to see tests in action.

```
HelloNFT Mint
    ✔ Should not throw BAD ADDRESS CHECKSUM error if correct recipient address is provided (1044ms)
    ✔ Should throw BAD ADDRESS CHECKSUM error if correct recipient address is provided (96ms)


  2 passing (1s)
```

2) The token identifier should be incremented after every successful mint
- The unit test in this scenario should verify that tokenId is correctly incremented after each successful mint operation.The test would look some what like follows:

```
it("Should check if tokenId is correctly increment after minting", async function () {
       const HelloNFT = await ethers.getContractFactory("HelloNFT");
       const helloNFT = await HelloNFT.deploy();
       await helloNFT.deployed();
      
       await helloNFT.mintToken("0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", "https://www.google.com");
       const ownershipRecord1 = await helloNFT.ownershipRecord("0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", 0);
       expect(ownershipRecord1[0]).to.equal(0)
      
       await helloNFT.mintToken("0xdD2FD4581271e230360230F9337D5c0430Bf44C0", "https://www.google.com");
       const ownershipRecord2 = await helloNFT.ownershipRecord("0xdD2FD4581271e230360230F9337D5c0430Bf44C0", 0);
       expect(ownershipRecord2[0]).to.equal(1)
});
```

- Again, run npx hardhat test:

```
HelloNFT Mint
    ✔ Should check if tokenId is correctly increment after minting (1163ms)


  1 passing (1s)
```

- In this article, we explored how to write unit tests for your Solidity smart contracts in the Hardhat environment. 
- Hardhat makes it really simple to build, compile, and test your contracts on the development chain and makes you confident while shipping and deploying.
- All-in-all, your test file hello-nft-test.js should look like follows.

```
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("HelloNFT Mint", function () {
   it("Should not throw BAD ADDRESS CHECKSUM error if correct recipient address is provided", async function () {
       const HelloNFT = await ethers.getContractFactory("HelloNFT");
       const helloNFT = await HelloNFT.deploy();
       await helloNFT.deployed();
      
       try {
           await helloNFT.mintToken("0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", "https://www.google.com");
          
           // If reached here, it means that code did not throw any error so we can simply pass the test
           expect(true).to.equal(true);
       } catch (err) {
           // If any error is thrown by code, in order for this test to pass,
           // we must check that error is not "bad error checksum"
           expect(
               'bad address checksum (argument="address", value="0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", code=INVALID_ARGUMENT, version=address/5.6.0)'
               ).to.equal(err.message);
           }
       }
   );

   it("Should throw BAD ADDRESS CHECKSUM error if correct recipient address is provided", async function () {
       const HelloNFT = await ethers.getContractFactory("HelloNFT");
       const helloNFT = await HelloNFT.deploy();
       await helloNFT.deployed();
      
       try {
           await helloNFT.mintToken("0x0000a0000a0aa00000aaa0aaa00a0a0a0a0a0000", "https://www.google.com");
          
           // If reached here, it means that code did not throw any error so we can simply pass the test
           expect(true).to.equal(true);
       } catch (err) {
           // If any error is thrown by code, in order for this test to pass,
           // we must check that error is not "bad error checksum"
           expect(
               'bad address checksum (argument="address", value="0x0000a0000a0aa00000aaa0aaa00a0a0a0a0a0000", code=INVALID_ARGUMENT, version=address/5.6.0)'
               ).to.equal(err.message);
           }
       }
   );

   it("Should check if tokenId is correctly increment after minting", async function () {
       const HelloNFT = await ethers.getContractFactory("HelloNFT");
       const helloNFT = await HelloNFT.deploy();
       await helloNFT.deployed();
      
       await helloNFT.mintToken("0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", "https://www.google.com");
       const ownershipRecord1 = await helloNFT.ownershipRecord("0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199", 0);
       expect(ownershipRecord1[0]).to.equal(0)
      
       await helloNFT.mintToken("0xdD2FD4581271e230360230F9337D5c0430Bf44C0", "https://www.google.com");
       const ownershipRecord2 = await helloNFT.ownershipRecord("0xdD2FD4581271e230360230F9337D5c0430Bf44C0", 0);
       expect(ownershipRecord2[0]).to.equal(1)
   });
   }
);
```
















































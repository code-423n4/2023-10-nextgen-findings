# NextGen Smart Contracts Analysis

## Description of NextGen
The NextGen project provides artists with the opportunity to explore AI Generative Art. This form of artistic expression involves integrating randomness, algorithms, geometry, and other elements to generate entirely distinct outcomes. The resulting artwork is quite literally, a partnership between the computer and the artist, utilizing various minting models that can be assigned to different phases. What's even more interesting in generative art is its focus on creating extensive series. Instead of aiming for a single, captivating artwork, a generative artist collaborates with an algorithm capable of generating tens, hundreds, or even thousands of artworks.
## 1. System Overview

### 1.1 Scoping

- [NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol): This contract includes the core functionality of the system such as creating collections, setting collections' data, updating collection info, updating images & attributes, updating metadata view of a collection, updating on-chain token data, artist signatures, collection freezing, and additional setter functions.
- [MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol): This contract includes the NFT minting functionality of the system, functions include setting collection costs and sales model, setting collection minting phases, airdropping tokens, minting tokens, burning, minting, auctioning, setting of primary/secondary splits for artists and teams, setting addresses and transferring collected funds.
- [NextGenAdmins.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol): This contract includes functionality related to registering admins such as function admins & collection admins, as well as checking admins.
- [RandomizerNXT.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol): This contract includes functionality for generating a random hash for each token during the minting process using NextGen's proposed approach.
- [RandomizerVRF.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol): This contract includes functionality for generating a random hash for each token during the minting process using [Chainlink's VRF service](https://docs.chain.link/vrf).
- [RandomizerRNG.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol): This contract includes functionality for generating a random hash for each token during the minting process using [ARRng.io's service](https://www.arrng.io/).
- [XRandoms.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol): This contract includes functionality used by the [RandomizerNXT](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol) contract, once called, it returns a random word from the existing word pool as well as a random number back to [RandomizerNXT](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol) which uses those values to generate a random hash.
- [AuctionDemo.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol): This contract holds the current auctions after `MinterContract::mintAndAuction` is invoked and allows users to bid on a token and claim the token after an auction finishes.

### 1.2 Access Control Roles
The 3 main contracts `NextGenCore`, `MinterContract` & `AuctionDemo` use a total of 4 access control modifiers for different mechanisms within the system.

- `FunctionAdminRequired`: This role has access to all the core functionality in the system incl. creating collections, setting/updating data and general changes such as setting new addresses and variables.

- `CollectionAdminRequired`: This role has access to functionality related to creating, setting up and maintaining collections within the system.

- `ArtistOrAdminRequired`: This role has access to functionality such as setting the artist's addresses.

- `WinnerOrAdminRequired`: This role has access to a sole function inside the `AuctionDemo` contract which is used to claim an auction once it's deadline is over. 

### 1.3 Access Restricted Functions

- The `NextGenCore` contract uses the `FunctionAdminRequired` and `CollectionAdminRequired` modifiers to restrict access control to some administrative functions such as [creating collections](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L130) and [setting collection data](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147), [adding a randomizer to a contract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L170), [updating collection info's](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L238) as well as some more updating/setting functions and a [collection freezing](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292) function.

- The `MinterContract` also uses the `FunctionAdminRequired` and `CollectionAdminRequired` modifiers, as well as an additional `ArtistOrAdminRequired` one. The following functions are restricted to the `FunctionAdmin`/`CollectionAdmin`'s: [setCollectionCosts](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L157), [setCollectionPhases](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L170), [airDropTokens](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181), [mintAndAuction](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L276), [updateDelegationCollection](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L302), [initializeBurn](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L308), [initializeExternalBurnOrSwap](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L315), [setPrimaryAndSecondarySplits](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L369), [acceptAddressesAndPercentages](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L408), [payArtist](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L415), and a few other set/update functions. Whilst, the following functions are restricted to the `ArtistOrAdminRequired`: [proposePrimaryAddressesAndPercentages](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L380) and [proposeSecondaryAdressesAndPercentages](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L394).

- Finally, the `AuctionDemo` contract uses only the `WinnerOrAdminRequired` modifier for the `claimAuction` function.

## 2. Architecture Overview & Diagram

[ARCHITECTURE OVERVIEW DIAGRAM](https://res.cloudinary.com/djwqwx2dp/image/upload/v1699869814/NextGen_Architecture_Diagram_ucg4xz.jpg)

The architecture in general seems to be solid, although I felt it could be improved upon. Some of the function architecture inside the `MinterContract` requires a "progressive" so to say, function invocation approach, for example in order to call `MinterContract::setCollectionPhases`, it requires `setMintingCosts[_collectionID] == true`, which requires you to to have beforehand called... `MinterContract::setCollectionCosts` - makes sense, and to call that, you need `gencore.retrievewereDataAdded(_collectionID) == true`, which in turn require you to have beforehand called `NextGenCore::setCollectionData`. The whole thing was a bit confusing at first but not necessarily a bad approach. Once I wrapped my head around it it made sense but I still needed to track which function was called when, to know in what state the contract currently is and if a `true/false` flag was passing or not.

The `mint` function inside `MinterContract` is very long and complex including multiple checks, if/else if's and it was challenging to grasp what is going on:
```javascript
function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        uint256 col = _collectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;
        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
            phase = 1;
            bytes32 node;
            if (_delegator != 0x0000000000000000000000000000000000000000) {
                bool isAllowedToMint;
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
                if (isAllowedToMint == false) {
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);    
                }
                require(isAllowedToMint == true, "No delegation");
                node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");
                mintingAddress = _delegator;
            } else {
                node = keccak256(abi.encodePacked(msg.sender, _maxAllowance, tokData));
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, msg.sender) + _numberOfTokens, "AL limit");
                mintingAddress = msg.sender;
            }
            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            phase = 2;
            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
            mintingAddress = msg.sender;
            tokData = '"public"';
        } else {
            revert("No minting");
        }
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
        for(uint256 i = 0; i < _numberOfTokens; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
        }
        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
        // control mechanism for sale option 3
        if (collectionPhases[col].salesOption == 3) {
            uint timeOfLastMint;
            if (lastMintDate[col] == 0) {
                // for public sale set the allowlist the same time as publicsale
                timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;
            } else {
                timeOfLastMint =  lastMintDate[col];
            }
            // uint calculates if period has passed in order to allow minting
            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
            // users are able to mint after a day passes
            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
        }
    }
```
Although I was unable to find bugs inside it, I believe very large and complex functions are prone to more bugs. In my opinion, a better approach would have been to split the logic into different minting functions depending on the needs.

Other than that, the architecture seems good for what it's intended. Nothing novel or crazy going on - does just what is needed for the job.
## 3. Codebase Assessment
- General Redability: The codebase demonstrated decent coding practice. Readability and organization could be improved. Clear variables and function names. 
- Code Style: The code follows a consistent style and indentation pattern as well as in-line comments guiding you through everything, but I felt like the comments could be improved to give more clarity for some of the large and complex functions inside the system. 
- Gas: Gas efficiency is generally maintained, although optimization of complex calculations and functions is advised. 
- Test Suite: The authors indicated a 100% test coverage suite, which I did find but I was not able to confirm as I am familiar and used to foundry, and the test suite is in hardhat.
- Errors & Events: This is a part that the codebase lacked. The event handling was rather limited and there were no `Custom Errors` present whatsoever.
- Upgradeability: Most parts of the system were able to be upgraded by the `Function Admin` incl. the `Core Contract`, `Minter Contract` and `Admin Contract`, which is always good practice.
## 4. Level of Documentation
The `Additional Context` part of the documentation inside the `NextGen` repo was something that I found great. They laid out a linear progression of the deployment process for each and every contract, how collections are set up for minting, information about the different trusted roles inside the protocol, how the randomizer contracts are intended for use, ideas for attacks, main invariants as well as some more additional informational points. 

A link for more detailed and intricate documentation was provided at their [website](https://seize-io.gitbook.io/nextgen/). The website documentation covers everything ranging from idea of the project, use cases, functionality, features, how to get started and how to apply as an artist. There were also 2 diagrams included which provide visual representation of the architecture as well as how to get started with creating a collection.

## 5. Systemic & Centralization Risks
- Complex Logic: As outlined earlier, the logic of the `minting` functions inside the `MinterContract` is so complex that they are naturally more prone to bugs, which should be avoided whenever possible. For core functionality like this, it would be wise for the protocol to conduct a follow-up audit after the report of this one comes out and the issues are all mitigated, in order to check if in turn, new ones have arised.

- Locked Funds Forever: I was also able to find what seems like a critical issue inside the `AuctionDemo` contract which proved easy to pull off and the impact was complete locking of user funds (in the form of auction bids). Although still an inconvenience, a soft-mitigation would have been if the `AuctionDemo` contract included an `emergencyWithdraw` function such as the `MinterContract` has. But since no such function exists to withdraw/save locked funds, the impact is high, as in that user bids could be locked inside the contract forever for each and every auction. Later I found out it is included in the bot-report and did not submit it.

- Centralization: As I have included in [1.3 Access Restricted Functions](#13-access-restricted-functions1.3), a huge part of the functionality of this contract is restricted to either the `FunctionAdmin` or `CollectionAdmin` roles. Some of the protocol's main points and ideas like creating collections, setting/updating their data etc. can only be done by a trusted admin. In my opinion some of the functionality could definitely be left to be decentralized if proper code architecture is maintained as to not run into issues.


## 6. Personal Findings

| Severity | Findings          |
|----------|-------------------|
| Critical | 0                 |
| High     | 1 (in bot-report) |
| Medium   | 1 (in bot-report) |
| Low      | 0                 |

## Time Spent ⌛

|            |            |
|------------|------------|
| Start Date | 07.11.2023 |
| End Date   | 13.11.2023 |
| Medium     | 7 Days     |
| Hours      | 30-40h     |

### Time spent:
35 hours
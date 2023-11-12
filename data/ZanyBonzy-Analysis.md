## **1. Codebase Reveiew**
### **1.1 General Overview**
  - NextGen aims to explore new directions in generative art and other non-art use cases for 100% on-chain NFTs.
  - It is a classic on-chain generative contract with extended functionality, including phase-based, allowlist-based, and delegation-based minting with a range of minting models, each of which can be assigned to a phase.
  - It also has the ability to pass arbitrary data to the contract for specific addresses to customize the outputs.

### **1.2 Scope and Architecture Overview**

#### **1.2.1 Contracts**
##### Сore contract 
  - [NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol) is the contract where the main functions of the protocols take place. It holds the basic functionalities of an ERC721 contract, as well as other setters and getters. Here, collections can be created, collection tokens can be minted/burned, aritsts can sign their collections and basic information about a collection or a specific token can be looked up. Artist royalty fee in bps can also be set. 
##### Minter contract
  - [MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol) is where various token mint requirements are set before minting. Here collection costs, [mint phases](https://github.com/ZanyBonzy/Code4rena-2023-11-NextGen/edit/main/AuditAnalysis.md#123-additional-info) and [sales models](https://github.com/ZanyBonzy/Code4rena-2023-11-NextGen/edit/main/AuditAnalysis.md#123-additional-info)), Information about artist's payments (splits, addresses, percentages etc) can be set and looked up. Funds are also transfered to the artists based on the previously set info.
##### Auction contract
  - [AuctionDemo.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol) is the contract used to auction off collection tokens. Here, would-be buyers can make bids, cancel their bids and claim their prize if eligible.
##### Randomizer contracts
  - The randomizer contracts all have the same function - generating a random hash for each token during minting. Here token hash is calculated using either the chainlink VRF ([RandomizerVRF.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol)), ARRNG randomizer ([RandomizerRNG.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol)) or the custom made NXT randomizer ([RandomizerNXT.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol)) which uses the [XRandoms.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol) contract to generate random numbers. 
##### Admin/access-control contract
  - [NextGenAdmins.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol) is the contract responsible setting up admin addresses. Here, addresses are registered as [global](https://github.com/ZanyBonzy/Code4rena-2023-11-NextGen/edit/main/AuditAnalysis.md#122-roles), [function](https://github.com/ZanyBonzy/Code4rena-2023-11-NextGen/edit/main/AuditAnalysis.md#122-roles) or [collection](https://github.com/ZanyBonzy/Code4rena-2023-11-NextGen/edit/main/AuditAnalysis.md#122-roles) admins. These admins have specific roles that they're allowed to perform in the protocol and are restricted by specific modifiers.

#### **1.2.2 Roles**
  - **Owner** - owns the protocol. His main function is to register a global admin.
  - **Global admin** - is in charge of and can perform every function that requires an admin.
  - **Function admin** - has access to specific functions in the protocol, mostly defined by the FunctionAdmin modifier.
  - **Collection admin** - is a member of the artist's team and is in charge of specific collections. His functions are quite limited and are defined by the CollectionAdmin modifier.
  - **Artist** - owns the collection. His functions are limited to signing the collections and updating his payment details.
  - All admin roles are trusted.

#### **1.2.3 Additional info**
- **Collection states** - A collection has two states - unfrozen and frozen. When a collection is frozen/locked, important collection info cannot be set or updated.
- **Sales/Mint models** - are the various models a through which collections can be sold to an interested party. The models are based on 4 major parameters, the `collectionMintCost`, `collectionEndMintCost`, `rate`, and `timePeriod`. Models include airdops, fixed price sale (costs are fixed, rate & time period are 0), exponential descending sale (cost decreases exponentially for a timeperiod till endcost is met, rate is set to 0), linear descending sale (costs decreases linearly based on set rate), periodic sale (1 token is minted at a time period with price increasing or staying stable per mint) and burn-to-mint (a NextGen 
 or non-NextGen token is swappped/burnt to get a new different NextGen token).
- **Sales/Mint phases** -  are the two main time periods when a token can be sold. An allowlist period, where an array of users are given first access to the tokens and a public period, where any interested users can buy the tokens.
  
## **2. Risks to protocol**
  - **Centralization Risks** - The protocol relies on the global, collection and function admins to perform specific tasks. They are trusted and are a points of failures for the protocol. 
  - **ThirdParty dependencies** - The protocol imports OZcontracts (albeit not by using the "import" tag). Any vulnerabilities in these imports can affect the contracts and pose a risk to the protocol's security.
  - **Randomness** - True randomness is hard to come by in solidity, and the randomizer contracts are to a point, good enough. Any issue with these sources of randomness can directly influence protocol token hash.
  - **Malicious users** - Malicious users can use the protocol as a dump ground by using the burn/swap-to-mint functions to swap stolen nfts for legitimate collections.
  - **Smart Contract vulnerabilities** - Flaws in the smart contracts also pose a risk to the protocol. 
    
## **3. Audit approach**
We approached the audit in 3 general steps after which we generated our report.
- **Documentation review & static analysis** -  We reviewed the readMe, the provided gitbook and explanations provided by the devs. We made notes of important point, invariants and asked questions for unclear parts. While this was going on, we ran the contracts through slither and compared the generated report to the bot reports. From this, we sorted out the known issues, false positives from the report.
- **Manual code review** - Here, we manually reviewed the codebase, ran provided tests, tested out various attack vectors. We looked for ways to break access control, break the main invariants, randomness and so on. We also tested out the functions' logic to make sure they work as intended and ensured that the contracts followed best programming practices. Here we noted out our the findings. 
- **Codebase comparisons** - After the manual review, we looked for protocols of the same type, compared their implementations and tried to find any repeated vulnerable patterns.

## **4. Final Thoughts**
  - The codebase is odd albeit well written. The team made a lot of unconventional decisions - for instance, the needed OZ contracts were copied as their own specific files rather than the importing whole package, admin functions were defined different from most protocols, not checking return values of low level calls, etc. For a next generation protocol, it does have some old school implementations.
  - The provided documentation is top class, really easy to understand and follow, but in contrast, the contracts are sparsely commented (on the plus side, they looked cleaner than if there had been NatSpec). Consider adding the required comments to the codebase, in accordance to NatSpec. It makes it easier for users and developers to easily understand the functions without having to keep looking them up in the gitbook. Also, there's no documentation on the AuctionDemo contract. Consider providing one.
  - Test coverage is 100% which looks good, however, it's a simple test file. A number of the large functions don't have invariant tests, no fuzzing tests. This should be fixed, as testing helps improve modularity, and helps catch basic bugs.
  - Error handling is fine, require errors are mostly used. Consider using custom errors instead, they're easier to maintain and are less gas intensive.
  - Input validation was practically non-existent. For instance, according to the devs, the public and allowlist sales should not be at the same time, collectionEndMintCost can be set higher than the collectionMintCost which defeats the purpose of a "descending" sale model. Yet there're no checks in place to prevent these from occuring. This issue gets a pass however, because most of the functions are called by "trusted" admins who are expected to not make a mistake. We recommend properly validating inputs, zero address checks in constructor etc. 
  - The OZ contract versions all seem to differ per contract. Some are based on the latest version 5.0, some as old as 4.4.1. We recommend updating all contracts to the latest version to make them match and to get the latest security upgrades.
  - We recommend making implementing an blocklist for stolen tokens as they can easily be burned/exhanged for legitimate tokens.
  - Ownable2step should be used instead of Ownable and functions to deregister admins (in case one goes rouge) should also be introduced.
  - FInally, we recommend taking a bit more time to reorganize the contracts, restructure the functions to follow CEI, mitigate discovered issues and implement constant upgrades/audits to keep the protocol up to date and secure.
    













### Time spent:
24 hours
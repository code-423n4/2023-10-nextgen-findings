# Approach taken in evaluating the codebase
During this 14-days audit the codebase was tested in various different ways. The first step after cloning the repository was ensuring that all tests provided pass successfully when run. After that began the phase when I got some high-level overview of the functionality by reading the code. There were some parts of the project that caught my attention. These parts were tested with Foundry POCs. Some of them were false positives, others turned out to be true positives that were reported. The contest's main page does a wonderful job providing main invariants that should never be broken and ideas for exploits. The [*gitbook*](https://seize-io.gitbook.io/nextgen/) was also a good resource especially when it came to better understanding the different sales models.

 

# Architecture recommendations
### Protocol Structure
Nextgen is a protocol that allows artist to create different collection utilizing the structure provided by Nextgen. There can be many different collections created with max 10_000_000_000 different NFTs that can be minted per collection. Each collection can have different artist, total supply, name, license, baseURi, library, description and collections script set in the NextGenCore contract. Additionally max collection purchases for each user can be set. The MinterContract is the contract responsible for minting the NFTs. Each collection can chose different sales model, specifying different sales structures. The protocol has been structured in a very systematic and ordered manner, which makes it easy to understand how the function calls propagate throughout the system. 


### What was unique for me?
 - The main purpose of Nextgen is to create generative collections and then freeze them. In most of the erc721 contracts admins can revert the base Uri. In Nextgen once the collection is frozen no-one, even the admins can’t revert the state. 
 - Creating separate collections with different characteristics from a single ERC721 contract.

# Codebase quality analysis
The structure of the project  is quite messy, all of the smart contract are placed in the same folder, including the interfaces. I would recommend to better structure the project, separate the interfaces, libs and contracts in separate folders. Group contracts that share purpose in the same folders. Instead of implementing (mainly coping) well known standards for which OpenZeppelin or Solady have battle tested implementations install the required libraries and import the required contracts. This leads to a much better structured project which is easier to navigate. This also pertains for the interfaces and libs. Nat spec comments are missing in the whole codebase. Some of the functions in the code contain a lot of logic in themselves, which adds a layer of complexity to the protocol. For example the mint function checks for different phases a sale can be in, then executes separate logic for each phase, additionally checking for the sale price model and if the price model is a periodic sale executes additional steps. Logic can be split in different functions, making the code more readable and easier for testing. There are no tests present in the codebase supplied for the audit, although sponsors have mentioned that there are tests. Although based on some of the vulnerabilities I discovered during this audit I guess only best case scenarios have been tested. I would recommend the the integration of fuzz and integrations tests.  

# Mechanism review
#### High-level System Overview
#### Chains supported
Ethereum Mainnet
#### Core Contract
The Core is the contract where the ERC721 tokens are minted and includes all the core functions of the ERC721 standard as well as additional setter & getter functions. The Core contract holds the data of a collection such as name, artist's name, library, script as well as the total supply of a collection. In addition, the Core contract integrates with the other NextGen contracts to provide a flexible, adjustable, and scalable functionality.

#### Minter Contract
The Minter contract is used to mint an ERC721 token for a collection on the Core contract based on certain requirements that are set prior to the minting process. The Minter contract holds all the information regarding an upcoming drop such as starting/ending times of various phases, Merkle roots, sales model, funds, and the primary and secondary addresses of artists.

#### NFT Auction
The Auction Demo contract purpose is to allow users to bid for a certain NFT for a specific time duration. There is no min price from which the bidding starts. Each user bids are not accumulated, if a user is outbid by 1 WEI, he will have to withdraw his bid and then bid again in order to become the highest bidder. The contract is completely broken containing multiple ways to be bricked or taken advantage of by malicious users. There are several things I would recommend the protocol to implement in order to fix the contract. First implement the Pull over Push pattern: https://fravoll.github.io/solidity-patterns/pull_over_push.html . Second instead of keeping all bids in array and looping trough it, making it very easy for a malicious actor to supply multiple bids with low amounts of WEI and brick the whole functionality. Implementing the Pull over Push pattern you can only store the highest bidder for a certain NFT. In order to easily track all bidders, combine each user's bid into a single bid, for example john bids 1 EHT, then bob bids 1.1 ETH and becomes the highest bidder, then john bids 0.2 ETH and becomes the highest bidder again. This way you won't have to loop over all bids every time a user wants to submit a bid, and user bids are accumulated. After an auction is over allow all the bidders that lost to execute a withdraw request and get their funds back. Implement the Push over Pull pattern for the address who put the NFT for auction, as this way if a malicious users wins the bidding (for example with a contract, to which an NFT can't be transferred), the address who put the NFT for auction can still withdraw the winning bid.

#### Randomizer contract
The Randomizer contract is responsible for generating a random hash for each token during the minting process. Once the hash is generated is sent to the Core contract that stores it to be used to generate the generative art token. NextGen currently considers 3 different Randomizer contracts that can be used for generating the tokenHash.
 - A Randomizer contract that uses the Chainlink VRF.
 - A Randomizer contract that uses the ARRNG.io service.
 - A custom-made implementation Randomizer contract.

There is not much else to be said about the Randomizer contracts, except that they contain systemic risks as two of the randomizer contract implementations depend on third party systems in order to work correctly. As for the custom implementation contract for randomness of the Nextgen team ```block.prevrando``` is not entirely safe method for randomness as there are couple possible attacks such as:
 - Selfish Validator Registrations: One could try to register especially advantageous validators that are chosen sooner and game the mechanism to receive a better NFT.
 - Bribing: A malicious user could also try to bribe validators in advance for blocks that might interest him in order e.g. censor some specific transactions or not propose a block at all.
But the possible gain of the attacker is nothing compared to the price he will have to pay to execute such an attack.
#### Admin contract
The admin contract and its roles are reviewed in the Centralization risks section.

# Centralization risks
#### Roles 
**1. Trusted Roles**
 - **Owner:** Is the deployer of the Admin contract, having the power to set and remove global admins. This is the address that will receive all the funds from the Minter contract in the case of an emergency withdraw.
 - **Global Admin:** Global Admin is set by the owner and have the power to call all functions except the function to register other global admins. He can register and remove Collection Admins, Function Admins
 - **Function Admin:** The Function Admin is set by the Global Admin, as he has the permissions to call functions for which he has been registered as an admin for via the specific function selector.
 - **Collection Admin:**
 - **Artist:**

**2. Normal Roles**
 - **Normal user:** A Normal User is any person that will use the Nextgen protocol. A normal user can mint and burn NFTs as well as participate in Auctions in order to acquire certain NFT.

#### Risks
The Admin contract is responsible for adding or removing global or function-based admins who are allowed to call certain functions in both the Core and Minter contracts. The Core and Minter contract call the Admin contract in order to check if a certain user is allowed to call a function. The Admin contract address can be changed by a Function Admin, to a contract where the owner is the Function Admin, but this will require the admin to be malicious. Owner, Global Admins and Functions Admins can damage the protocol and the normal users including the collection artists in different ways. The code base is heavily centralized but the sponsors have mentioned that the trusted admins will be trusted gnosis safe wallets with 2/3 or 3/4 signatures so they will check if a request is malicious  or not. Compromising the trusted addresses will be extremely difficult. That said users of the protocol can never be 100% sure that the owners or admins won't rug pull them, in the case of Nextgen this is also true for the artists of the collections as all funds from the minting contract can be withdrawn by the admins via the emergencyWithdraw.




### Time spent:
25 hours
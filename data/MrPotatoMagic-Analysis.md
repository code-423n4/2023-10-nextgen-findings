# Analysis Report

## Preface

This audit report should be approached with the following points in mind:

 - The report does not include repetitive documentation that the team is already aware of. It does include suggestions to provide more clarity on certain aspects in the documentation.
 - The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
 - If there exists repetitive documentation (mainly in [Mechanism Review](#mechanism-review)), it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

## Approach taken in evaluating the codebase

### Time spent on this audit: 14 days (Full duration of the contest)

Day 1-4
 - Understand the idea of the project and grasp a high level mental model of the contract interactions.
 - Reviewed the NextGenAdmins, NextGenCore and MinterContract to understand the core functionality of the protocol.
 - Add inline bookmarks in the code in the form of questions, such as, "Could X do Y to mint more tokens or bypass this Z check?"
 - Noted down some obvious gas optimizations and low-severity issues not covered by the bot

Day 5-9
 - Reviewed the AuctionDemo and the 3 Randomizer contracts
 - Explored findings and manually validated them with inline bookmarks
 - Read about Chainlink VRF security consideration as well as researched the ARRNGController.sol contract deployed on Ethereum mainnet.
 - Added some more notes for attack ideas to visit later on

Day 10-11
 - Writing Gas/QA report
 - Writing HM reports with only written POC for now

Day 12-14
 - Setting up a foundry environment within the hardhat tests
 - Validating most of the HMs with coded POCs/tests
 - Filtering out invalid and low-severity issues from HMs
 - Writing Analysis Report

## Architecture recommendations

### Protocol Structure

The protocol has been structured in a very simplistic manner. The contracts in the protocol can be divided into five categories, namely, Core, Minter, Admin, Auction and Randomizer contracts. 

![](https://user-images.githubusercontent.com/109625274/282527537-9b0dddfb-0110-47cd-9f29-74a861aa5a45.png)

### What's unique?
1. Use of Solidity's rounding for Periodicity in Sales model 2 and 3 - Solidity's loss of precision/rounding down has been used as a feature to calculate tDiff and periods for the Exponential/Linear descending Sales (sales option 2) and Periodic Sale model (sales option 3). This allows the property of periodicity to exist in the codebase, allowing to enforce invariants such as "1mint/period".
2. Onchain and offchain metadata compatiblity - The NextGen contracts allows collections of any type to support onchain and offchain metadata compatibility, which opens up doors to experimental features such as University Certificates, Onchain Music, Artwork and many more sectors to flourish.
3. Freezing and setting final supply for collections - Most NFT contracts allow changing the supply by minting/burning tokens, thereby fluctuating the demand. By locking the data and final supply, immutability is revitalized into the collection, which can help increase user trust into the creators and their collection.
4. Intentional Pseudo-Randomness - The RandomizerNXT.sol contract introduces a model similar to PRNG, which provides the NextGen team to sell it as a separate service as well as provide the collection artist with more options, ideas and innovations to integrate with their existing collection idea/generative script.
5. Connection with the outside NFT world - Other than marketplaces where NFT's can be directly transferred/bought/sold, NextGen introduces another market to exist using their burnOrSwapExternalToMint() function in MinterContract with external NFT tokens. This is a particularly simple implementation but opens up a lot of pathways, considering NextGen collections are experimental and can include collections ranging from onchain music ownership to mintpass systems.
6. Combination of 3 Sale models with allowlist/public phases - There are currently numerous combinations between the two phase types and 3 sale models, which allow artists to hold multiple allowlist/public phases with any sales model of their choice. For example, when the NFT trade market (such as [nftperp](https://nftperp.xyz/)) is in a bull run, the collection admins can generate more revenue by selling more tokens to users by setting up multiple rounds of allowlist/public phases ready for minting.
7. Guarded launch of collections through max allowances - The collection admins can use max allowances along with periodic sale minting to ensure the prices of their NFTs (in the external markets) are less volatile during launch. This is a hidden but unique feature of allowances along with periodicity that unlocks new opportunities for the collection creators.

### What ideas can be incorporated?

 - Integrating ERC1155 tokens to not limit artist to ERC721 standard non-fungiblity model
 - Allowing users to auction their NFT tokens
 - Allowing support for multi-artist collections since the current codebase only allows one artist's signature to be stored for a collection
 - Enabling delegatees to enter auction on behalf of delegator
 - Support for re-auctioning a token in AuctionDemo.sol

## Codebase quality analysis

Since I found numerous coded POC confirmed issues, the only way I would decide the quality of this codebase is through the issues per contract and design structure i.e. code readability/maintainability.

**Admins Contract:**
The NextGenAdmins contract was quite simplistic. I had a quick glance at it since it was short and brief to understand. Since there were no issues here, I would mark this as high-quality.

**Randomizer Contracts:**
The RandomizerVRF.sol, RandomizerRNG.sol and RandomizerNXT.sol collectively only had around 3 issues, some of which had no severe impact on the codebase. When it comes to the security considerations and code maintainability, the team seems to knowing the intracacies and some potential attack vectors, which have been neatly mitigated with access controls and limiting function visibility. Due to this, I would mark this as high-quality.

**AuctionDemo Contract:**
This contract has 3 High-severity issues, which were missed out due to loose equality checks and missing storage updates in the claimAuction() and cancelAllBids() functions. The design choice of participate-claim-cancel makes sense but the severity of the issues anyday overshadow the design choice for quality since the potential impact of the issues expand not only to the current auction but also to other auctions running in parallel. Due to this, I would mark this as low-quality.

**MinterContract:**
Most of the high and medium-severity issues I found were heavily related to this contract's missing state updates, require checks, missing functionality for support of sales model 3 as well as breaking of invariants. Due to this, I would mark this as low-quality.

**NextGenCore Contract:**
The NextGenCore contract included 2 High-severity issues, one from reentrancy through _safeMint() and the other due to missing state update in burnToMint(). The rest of the contract does not seem to have any major issues that I've come across. Due to this, I would mark this as low-quality.

**Test Coverage of these contracts**
The Core contracts have tests implemented in hardhat except AuctionDemo.sol and the Randomizers. The core contract have not been tested for specific branches or combinations but only basic functionality. Additionally, no fuzz testing, strict invariant testing or fork testing has been performed, considering the fact that randomness services like Chainlink VRF is used.

Although I have not delved deeper into the issues here, this is my overall opinion on the codebase quality analysis based on both issues per contract and code readability/maintainability. Due to this, I woould rank it as almost close to touching the Medium-quality bar but no higher.

## Centralization risks

A. There are 4 trusted roles that are already mentioned in README, which are (from highest to lowest power):
1. Global Admin
2. Function Admin
3. Collection Admin
4. Artist

B. Each of these roles have their downsides. But let's see how they can be mitigated:
1. In case, the collection admin and artist turn out to be malicious, the global admin can just revoke their permissions. 
2. But in case, the global and function admins turn out be malicious, the collection admins and artists are at risk.
3. In order to ensure less centralization in this aspect, each collection admin/artist should have a separate multi-sig with the global/function admin. But unfortunately, this will not be enough since the global/function admins can always gain majority or remove collection admins/artist. In order to further decentralize this, consider implementing a RBAC timelock on all admin related functions, which can be voted upon by the collection admins/artists. This would be the safest implementation in my opinion.

C. Last but not the least, there is another crucial role in the codebase, which has not been surfaced even in the README. This is the ownerOf() or the deployer of the AuctionDemo.sol contract. This role is entrusted with or receives all the auction winner's ETH bid amounts. It is important to ensure that this address is some sort of multisig handled by the team.

Overall, the codebase is reasonably centralized since certain functions requiring it to be but there are inconsistencies in how these access controls have been applied on functions (especially in the NextGenCore contract). I would highly recommend the sponsors to create a deeper guide on what actions each role in the hierarchy are allowed to perform.

## Resources used to gain deeper context on the codebase

1. [NextGen documentation](https://seize-io.gitbook.io/nextgen/)
2. [Chainlink VRF - Security Considerations](https://docs.chain.link/vrf/v2/security)
3. [ARRNGController contract live on Ethereum mainnet](https://vscode.blockscan.com/ethereum/0x000000000000f968845afB0B8Cf134Ec196D38D4)

## Mechanism Review

### High-level System Overview

![](https://user-images.githubusercontent.com/109625274/282560390-c23260f6-755f-4bb5-b9f1-4f9bc679149c.png)

### Chains supported
 - Only Ethereum is supported currently

### Documentation/Mental models

#### Here is the basic inheritance structure of the contracts in scope

**Note: The NextGenCore, MinterContract, NextGenAdmins and AuctionDemo contract are singular and independent of each other and do not use any form of inheritance amoung themselves. This I believe is a good design choice since it allows separation of storage and concerns among each other.**

The RandomizerNXT.sol is the only contract that inherits from another in-scope contract i.e. XRandoms.sol

![](https://user-images.githubusercontent.com/109625274/282554425-a22727f5-b607-40cd-80c6-f371db28afc8.png)

### Features of Main Contracts

![](https://user-images.githubusercontent.com/109625274/282552920-938475e8-0bda-4353-b4db-691b8923f549.png)

### Features of each Sales model

Periodic Sale (Option 3)
 - Mints are limited to 1 token during each time period (e.g. minute, hour, day).
 - The minting cost can either stable or have a bonding curve (increase) over time.
 - If `rate = 0`, minting cost price does not increase per minting.
 - If rate is set, then the minting cost price increases based on the tokens minted as well as the rate.

Exponential Descending Sale (Option 2):
 - At each time period (which can vary), the minting cost decreases exponentially until it reaches its resting price (ending minting cost) and stays there until the end of the minting phase.
 - In this model the starting and ending minting costs and the time period are set while the rate is 0.

Linear Descending Sale (Option 2):
 - At each time period (which can vary), the minting cost decreases linearly until it reaches its resting price (ending minting cost) and stays there until the end of the minting phase.
 - In this model all parameters need to be set.

## Systemic Risks/Architecture-level weak spots and how they can be mitigated

1. Using ARRNG as a service for randomness

I did some past transaction research on the ARRNGController.sol contract live on Ethereum mainnet and found out there has only been one call of requestRandomness and serveRandomness, which was called almost 100 days back ([see here](https://etherscan.io/address/0x000000000000f968845afB0B8Cf134Ec196D38D4)). Since there is no past transaction data to study, the reliability of the service cannot yet be guaranteed.

Furthermore, if you look closely, it took 8 block confirmations for the serve randomness to be called through the RNG service. This would be necessary if we were on POW but now the Ethereum POS network will work just fine with 3 minimum block confirmations ([as done over here in the RandomizerVRF.sol contract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L28)) due to a block requiring 2/3rd of the total staked ether votes in favour in order for it to become justified. Thus, using only Chainlink VRF might be the best option while still ensuring the fastest response time.

![](https://user-images.githubusercontent.com/109625274/282561975-3b334247-d33f-4d74-8a1e-826b2fc690a7.png)

2. Using OZ's AccessControl contract - Currently there are no issues with the Admins contract and never might be in the future but it is good practice to use an existing standard such as AccessControl.sol by OpenZeppelin due to its widespread usage and modularity for handling roles in all types of situations.

3. Although addressed in the bot report, there are very high changes of the current implementation causing a DOS issue with airdropping tokens since external calls are made to the gencore contract in a for loop. To mitigate this, ensure airdrops are done in batches such that the block gas limit is not exceeded if airdropping to many users.

4. Frontrunning in mint() - The mint() function in the minter contract is prone to frontrunning by bots to mint tokens earlier. This would become a major issue for a free or low cost collection that uses the Periodic Sale model (sales option 3), which only allows 1mint/period. This would affect the normal users who will not be able to mint() unless the bot lets them for a certain time. This can only be mitigated through Flashbots RPC in my opinion but raising this issue in case sponsors find some leads to solutions.

5. Miners manipulating block.timestamp - It is possible for miners to manipulate the block.timestamp by a few seconds in order to mint() first during a Periodic Sale (sales option 3). The impact as mentioned previously is the same since normal users are affected in these edge case scenarios.

6. lastMintDate stores incorrect time - In one of my issues, I have demonstrated how it is possible for lastMintDate to be set to a very far time in the future if a collection uses Periodic sale during both allowlist and public phases. This is a big system risk to the protocol since it breaks the 1mint/period invariant. The issue includes a coded poc as well which demonstrates the issue and its impact on the users and creators of the collection.

7. Randomness is lost - It is possible for the VRF subscription plan to run out of LINK or the RNG contract to run out of ETH. Due to this, any randomness pending to be received by the Randomizer contracts is lost and the tokenHash of the tokenId is left empty forever. To avoid such an issue, implement a bot or emergency mechanism that alerts the team when the LINK or ETH goes below a certain critical threshold.

8. There are several inconsistencies between where the NFTDelegation mechanism is used and where it is not. This ambiguity denies the cold wallet users using delegation as a mechanism to avoid signing risky transactions and participating in certain services such as auctions, burnToMint() etc. Ensure the NFTDelegation mechanism is implemented in all places where normal hot wallet users would interact as well.

9. Incomplete comment can lead to problems during phases - This [comment](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L243) here does not mention to set the allowlistEndTime to 0. If this is not set to 0 and the collection admins think allowlistEndTime also has to be set to publicEndTime, this would cause the public phase calls to enter the allowlist if conditional block in the mint() function first, leading to incorrect behaviour. Make sure to provide clear documentation on how values for phases can be set correctly and with more clarity in order to avoid misconfigurations.

10. In AuctionDemo.sol, it is possible that no bids can be placed on an auctioned token (highly unlikely but still possible). In such a case, re-auctioning the token is not supported by the AuctionDemo.sol contract, which would be a problem. Though this can be solved by deploying another auction contract, adding a re-auctioning functionality will make the codebase more verbose and resilient in case of such edge scenarios.

11. Limitation of token payment methods when minting - Try integrating more ERC20 tokens and stablecoins in the current model since only ETH is accepted currently.

## Some questions I asked myself

1. Can a user prevent another user from setting a higher bid?
   - No

2. Can updating the core contract in minter contract cause any issues?
   - Yes, it will prevent collections on the old core contract to not be allowed to use airdrop(), mint() and burnToMint() functionlity.

3. If a collection is created, can one mint before randomizer contract or any other contract is set?
   - No call would revert since calculateTokenHash() does not exist.

4. Can minting start before data is set or artist signature is set?
   - As long as dates are set yes, but this would require admin misconfig

5. How many collections can be supported based on how Ids are served in the gencore contract?

 - To get a sense: Gauging the number of collections and number of tokens per collection that can exist with the current implementation of reserved indexes:
Number of collections = type(uint256).max / 10000000000 = 115792089237316195423570985008687907853269984665640564039457584007913129639935 / 10000000000 = 11579208923731619542357098500868790785326998466564056403945758400791 collections
Number of tokens per collection = 10000000000

 - Suggestion for sponsors: Number of tokens per collection can be increased since the NextGen contracts is experimentational and it's growth should not be underestimated in case the concept of university certificates (issued each year) or some new idea pops up that requires more than the current reserved index range of 10000000000 to be used.

6. Can we reenter through cancelBid? 
   - Not possible since auctionDataInfo is updated to false before making external call to return bidder's ETH

7. Small mistake in docs

 - Mistake in docs - https://seize-io.gitbook.io/nextgen/for-creators/sales-models#sales-models-examples 

 - In linear descending model, the statement is incorrect. It should be During 10th Price period price 3.1 ETH and 11th Price period price remains constant as shown in te diagram above the explanation as well

### Time spent:
140 hours
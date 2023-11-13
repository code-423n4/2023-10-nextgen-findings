# Analysis

## Table of Contents

- [Approach](#approach)
- [Architecture Overview](#architecture-overview)
- [Testability](#testability)
- [Centralization Risks](#centralization-risks)
- [Systemic Risks](#systemic-risks)
- [Summary of Reported H/M findings](#summary-of-reported-hm-findings)
- [Summary of Reported QA findings](#summary-of-reported-qa-findings)
- [Learnt About](#learnt-about)
- [Other Recommendations](#other-recommendations)
- [Security Researcher Logistics](#security-researcher-logistics)
- [Conclusion](#conclusion)

## Approach

The auditing process commenced with a high level but brief overview of the entire [NextGen smart contracts](https://seize-io.gitbook.io/nextgen/) ecosystem using the docs as a guide, this wasn't restricted to only the contracts in scope.

This was then followed by a multiple, meticulous, line-by-line security review of the sLOC in scope, beginning from the contracts small in size like `XRandoms.sol` ascending to the bigger contracts like the `MinterContract.sol`.

The concluding phase of this process entailed interactive sessions with the developers on Discord. This fostered an understanding of unique implementations and facilitated discussions about potential vulnerabilities and observations that I deemed significant, requiring validation from the team.

## Architecture Overview

| Contract                                                                                                         | Source Lines of Code (SLOC) | Description                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [XRandoms.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol)             | 39                          | Assists RandomizerNXT by providing random words and numbers for the hash generation process.                                                                                                                                                                                                                                                                                                                        |
| [RandomizerNXT.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol)   | 51                          | Generates unique hashes for each token during minting using a proprietary method developed by NextGen.                                                                                                                                                                                                                                                                                                              |
| [NextGenAdmins.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol)   | 61                          | Handles administrative roles, managing both global and function-specific admin permissions for Core and Minter contracts.                                                                                                                                                                                                                                                                                           |
| [RandomizerRNG.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol)   | 72                          | Generates token hashes randomly using the ARRng.io service.                                                                                                                                                                                                                                                                                                                                                         |
| [RandomizerVRF.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol)   | 87                          | Utilizes Chainlink's VRF service to generate random hashes for tokens.                                                                                                                                                                                                                                                                                                                                              |
| [AuctionDemo.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol)       | 114                         | Manages auction processes, enabling user bidding on tokens and facilitating the transfer of tokens to the highest bidders.                                                                                                                                                                                                                                                                                          |
| [NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol)       | 366                         | Central contract for ERC721 token minting, incorporating standard ERC721 functionalities and additional setter & getter functions. This contract holds the data of a collection such as name, artist's name, library, script as well as the total supply of a collection. In addition, the Core contract integrates with the other NextGen contracts to provide a flexible, adjustable, and scalable functionality. |
| [MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol) | 475                         | Mints ERC721 tokens for collections, following specific pre-defined conditions. This contract holds all the information regarding an upcoming drop such as starting/ending times of various phases, Merkle roots, sales model, funds, and the primary and secondary addresses of artists.                                                                                                                           |

### Important Links

- **Documentation:** [NextGen Documentation](https://seize-io.gitbook.io/nextgen/)
- **Website:** [Seize.io](https://www.seize.io/)
- **Twitter:** [6529 Collections](https://twitter.com/6529Collections) / [punk6529](https://twitter.com/punk6529)
- **Discord:** [NextGen Discord](https://discord.gg/nQDAkHt3)

> Graphic representation on how everything works attached below

![](https://pbs.twimg.com/media/F93Q4o_WsAAzKHw.jpg)

## Testability

The system's testability is commendable being that is has wide coverage, one thing to note would be the occasionally intricate lengthy tests that are a bit hard to follow, this shouldn't necessarily be considered a flaw though, cause once familiarized with the setup, devising tests and creating PoCs becomes a smoother endeavour. In that sense I'd commend the team for having provided a high-quality testing sandbox for implementing, testing, and expanding ideas.

## Centralization Risks

The centralization risks attached to this protocol is off the roof, this is cause protocol has stated that **all parties** are trusted, the hosts, artists, function admins, collection admins, every single one, which means that if I am to start listing areas where these could be an issue, this would be pages of submissions, as such I'd just list two:

- `emergencyWithdraw()` could be an avenue **to steal all the tokens** in any contract where it's been employed
- The `acceptAddressesAndPercentages()` is pretty up there when it comes to one of the risky part of the protocol, an admin can always just call this and set the statuses as false effectively breaking access to `payArtist()` since it requires the statuses of both the primary and secondary addresses to be true.

## Systemic Risks

- Once the `freezeCollection()` executed the collection's freeze status becomes true and cannot be altered... this seems to be a very risky way to operate.
- hardcoding the receiver of the funds from `emergencyWithdraw()` is a flaw as the key to this account could be lost which would lead to all funds attempted to be emmergency withdrawn actually been lost, since thereis alreadya n admin check allow the caller to pass in the address to receive the funds.

## Summary of Reported H/M findings

During my security review, I identified several High/medium-severity findings impacting the smart contract functionalities, each presenting unique challenges and risks to the system's overall integrity and compliance.

The main highlight and most important one was found in the `auctionDemo.sol` contract, which was a significant flaw in regards to the `claimAuction` function. This results in all bidders' funds sent for bids, including the highest bidder's, being locked in the contract, and the NFT token intended for the highest bidder also remains stuck. Moreover, the owner cannot receive the winning bid funds as the auction can never be successfully claimed. This issue aligns with various attack ideas outlined for auctions, including the token not transferring to the winning bidder, funds not being refunded to the bidders that didn't win, and the owner not receiving the highest bid funds post-bidding process. The root cause is the use of `IERC721.safeTransferFrom()` in the `claimAuction` function, which could revert if the highest bidder's address is incapable of receiving ERC721 tokens. To address this, it's recommended to replace `safeTransferFrom()` with `transferFrom()` in the `claimAuction` function and to clearly document the requirement for participants to be able to interact with ERC721 tokens.

A potential medium issue arises in the `cancelBid` function, where even if a user is in their right to cancel bids, they might still be forced to receive the NFT tokens. This occurs because both `cancelBid` and `claimAuction` functions can execute at the same timestamp, _a rare case, I know_ but seems to stem from a logical flaw. Since this leads to potential logical conflicts and impeding users' decision-making. To resolve this, it's advisable to adjust the `cancelBid` function to not accept execution whenever `claimAuction` can also be executed.

Lastly, the implementation for requesting randomized numbers from Arrng in the smart contract is flawed, with a significant impact on the contract's functionality. The issue lies in the constant `ethRequired` set by the admin for requesting randomness, which doesn't align with the fluctuating cost of randomness requests from Arrng. As a result, transactions might not go through if `ethRequired` is less than the required `msg.value`, leading to a denial of service. Since the cost of randomness requests isn't constant(_during the contest, I noticed the cost go up almost 3x in price as attached with images in the issue's report_), so `ethRequired` might not always `>=` the required amount, it's recommended to use `address(this).balance` for each query to ensure successful transaction execution.

## Summary of Reported QA Findings

The `NextGenRandomizerVRF` contract, post-initial deployment, all queries to Chainlink's VRF would inevitably revert due to an incorrect `keyHash` setting, specifically not aligned for the Ethereum mainnet. This misconfiguration, although presenting a low impact since it's rectifiable even after the project goes live, currently renders all Chainlink queries inoperative. The core of the issue lies in the `RandomizerVRF.sol`, where the `keyHash` is set to `0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15`. This particular `keyHash` is suitable for the Goerli testnet rather than the Ethereum mainnet, as evidenced by Chainlink's documentation which clarifies that `keyHash` represents the gas lane for maximum gas price adjustment. The discrepancy can be further confirmed by reviewing the supported gas lanes, where it becomes evident that the set `keyHash` is not intended for mainnet use. To address this problem and ensure functional integrity from the outset, it's recommended that the contract be initialized with a valid `keyHash` appropriate for the Ethereum mainnet.

The `min()` function has an oversight, it originally was designed to limit minting to one token per predefined time period, but that's not been followed, which could inadvertently inflate the token supply per period wise, contradicting the contract's documented tokenomics. This issue arises when more than one period elapses without minting. For instance, if a period is set to one minute and ten minutes pass with no minting, the `lastMintDate[col]` would be outdated, reflecting a time ten periods ago. The current implementation, which does not update `lastMintDate[col]` to the current timestamp, relies instead on `viewCirSupply`, paving the way for multiple tokens to be minted in the same period. To address this, it is recommended that the `mint` function be revised to update `lastMintDate[col]` with the current block timestamp immediately after a successful mint. This will ensure that `tDiff` is always calculated from the most recent minting event, thus enforcing the "one mint per period" rule in line with the contract's design.

Another issue concerns the `emergencyWithdraw()` function, which has hard-coded the receiver of the funds to the owner of the `adminsContract`. This design choice poses a risk of funds being permanently lost if the private key to the admin account is ever lost or compromised. Moreover, the function, intended for quick and safe withdrawal of all contract funds in emergencies, might be compromised due to its reliance on external calls, which introduce points of failure. These external calls could lead to the function's reversion, especially if influenced by the contract's owner or due to implementation errors. To enhance the effectiveness and reliability of `emergencyWithdraw()`, it's recommended to modify the function to accept the recipient's address as a parameter, thereby allowing the calling admin to specify the withdrawal destination, and to eliminate dependencies that might restrict its execution, as exemplified by the Pancakeswap Smart Contract's approach.

In the `NextGenRandomizerNXT` contract, there exists an inconsistency in the implementation of the `updateAdminsContract()` function, particularly when compared to its counterpart in the `NextGenRandomizerRNG`. The `NextGenRandomizerNXT` version lacks a crucial `require` check that ensures the new contract address actually corresponds to an admin contract, a safeguard present in the `NextGenRandomizerRNG` implementation. This discrepancy potentially leaves `NextGenRandomizerNXT` vulnerable to unintentional or malicious updates to its admin contract, posing a risk to the contract's integrity and security. Specifically, the `NextGenRandomizerNXT.updateAdminsContract()` method does not validate if the given contract is indeed an admin contract, as opposed to the `NextGenRandomizerRNG` version, which includes a `require` statement for this purpose. To mitigate this risk, it is recommended to introduce the same validation check from the `NextGenRandomizerRNG` to the `NextGenRandomizerNXT` in the `updateAdminsContract` function, ensuring consistency and enhanced security across both implementations.

Additionally, there's an issue of a redundant multiplication operation in the `burnOrSwapExternalToMint` function of the contract, where the line `require(msg.value >= (getPrice(col) * 1), "Wrong ETH");` unnecessarily multiplies the price by 1. This redundancy doesn't impact the functional outcome but leads to slightly increased gas costs and diminished code readability. To streamline the contract and potentially save on gas costs, it's recommended to revise the `require` statement by removing the multiplication by 1. The updated line should read `require(msg.value >= getPrice(col), "Wrong ETH");`, thereby enhancing clarity and efficiency of the code.

In the NextGen Admin Contract, the `registerBatchFunctionAdmin` function currently presents operational inefficiencies and increased transaction costs due to its design. The function uses an iterative method to set the status for each function selector, but this requires multiple contract function calls to assign different statuses (true/false) to various selectors. This design can be optimized by modifying the function to accept an array of bools (`bool[] memory _statuses`) instead of a single `bool _status`, allowing for individual status assignments per selector in a single transaction. Implementing this change necessitates adding a check to ensure that the lengths of `_selector` and `_statuses` arrays are the same, avoiding potential mismatches and errors.

Another issue noted is in the `updateRNGCost` function, where the value of `ethRequired` is updated without checking if it's different from the existing value. This oversight can lead to unnecessary transactions and inefficiencies. The recommended mitigation step is to include equality checks in setter functions to ensure that the value is updated only if it differs from the current one, thus improving the code structure and reducing unnecessary operations.

The `updateImagesAndAttributes` function in its current implementation risks inefficiency and inconvenience, as it reverts the entire transaction if any one of the provided token IDs belongs to a frozen collection. This design could cause significant issues for administrators when updating images and attributes for multiple tokens, especially if they inadvertently include tokens from frozen collections. To address this, it's recommended to replace the `require` statement within the loop with an `if` statement that simply skips the frozen collections and continues updating the rest. This change would allow for more efficient and user-friendly updates, reducing the risk of transaction failures due to a single frozen collection token.

In the `auctionDemo.sol` smart contract, the `returnHighestBid` function uses an iterative method to find the highest bid for a given token, which is redundant given the `participateToAuction` function's design that ensures each new bid surpasses the previous ones. This unnecessary iteration could lead to increased operational overhead and transaction costs. The `returnHighestBid` function iteratively searches for the highest bid, even though the highest bid will always be the last item in the `auctionInfoData[_tokenid]` array due to the checks in `participateToAuction`. To streamline the process and enhance efficiency, it's recommended to directly retrieve the last bid from this array instead of using a loop, as the last bid is guaranteed to be the highest.

Another issue arises in the `updateCollectionInfo` function in the NextGen smart contract, which contains a potential out-of-bounds vulnerability. The function lacks a crucial index range check when updating, potentially leading to errors if an out-of-bounds index is provided. Specifically, there is no check to ensure that the `_index` provided is within the valid range of `collectionScript`, creating a risk of unintended behavior. To mitigate this, it's suggested to include a range check for `_index` before updating `collectionScript`, ensuring it falls within the valid indices of the array.

Lastly, the `airDropTokens` function in the same contract lacks proper documentation, leading to potential misunderstandings or misuse of the function. The function is crucial for airdropping tokens but is not adequately explained in the [official documentation](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/core), unlike other functions such as `mint` and `burn`. To address this, the documentation should be updated to include detailed information about `airDropTokens`, covering its purpose, parameters, expected behavior, restrictions, use cases, and any potential risks. This update will provide users and developers with a clearer understanding of the function's role and usage within the contract's ecosystem.

## Learnt About

During the course of my security review, I had to research some stuffs to understand the context in which they are currently being used, some of which led to me learning new things and having refreshers on already acquainted topics. Some of these topics are listed below:

- `EIP-2981`.
- Using the `functionSelector` method to grant access to accounts for some functions.

## Other Recommendations

### **Enhance Documentation Quality**

I'd like to start by commending the quality of the [docs](https://docs.partydao.org/). They are generally well-crafted. However, there are some gaps concerning what's within the scope, as already submitted in the QA report, some functions are not present in the docs whcih isn't a good practise.

### **Improve Testability**

Whereas the previous comment on testing was commending it, do note that a few contracts do not have any tests whatsoever, which is not good practise and should be fixed, an example is the Auction contract

### **Onboard More Developers**

Having multiple eyes scrutinizing a protocol can be invaluable. More contributors can significantly reduce potential risks and oversights. Evidence of where this could come in handy can be gleaned from codebase typos, another reason why is the availability of the developers during the auditing process. i.e how hard it was to get a hold of developers on discord to talk about potential bug cases.

### **Leverage Additional Auditing Tools**

Many security experts prefer using Visual Studio Code augmented with specific plugins. While the popular [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) has already been integrated with the protocol, there's room for incorporating other beneficial tools.

### **Enhance Event Monitoring**

Current implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. A shift towards a more purposeful utilization of events is recommended.

### **Refine Naming Conventions**

There's a need to improve the naming conventions for contracts, functions, and variables. In several instances, the names don't resonate with their respective functionalities, leading to potential confusion, in other cases clear typos have been made in the names like the function `participateToAuction` could be `participateInAuction` instead.

## Security Researcher Logistics

My attempt on reviewing the Party Protocol spanned around 30 hours distributed over 3/4 days:

- 1 hour dedicated to writing this analysis.

- 2 hours were allocated for discussions with sponsors on the private discord chat regarding potential vulnerabilities.

- 2 hours over the course of the 4 days _(~30 minutes per day)_ was spent lurking in the public discord group to get more ideas based on explanation provided by sponsors to questions other security researchers have asked.

- The remaining time was spent on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports._

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, due to the fact that it has a little amount of nSLOC and how all parties are trusted, most _easy to catch_ contextual bugs have already been mitigated by stating that all admins are trusted, nonetheless it was a fun ride seeing the unique implementations employed in the codebase.


### Time spent:
27 hours
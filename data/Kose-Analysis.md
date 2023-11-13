# 1. General Overview

The Nextgen Protocol allows artists to generate and publish NFTs using generative scripts on the blockchain. It offers various minting features like airdropping and mintpassing, alongside diverse sales models and auctions. This protocol isn't just beneficial for Web3 artists; it can also aid organizations, such as universities and companies, in creating blockchain-based certificates. While the protocol's primary use might not be for creating certificates, it is one of them, and bringing these organizations onto the blockchain is a significant achievement. However, the protocol's codebase is highly centralized and susceptible to various admin-related issues, including trust and admin errors. This response will address several issues related to admin functions, as it's essential to consider the protocol's various uses. Even if Nextgen admins are expected to be experienced in Web3 and less likely to make mistakes, other protocol deployers may not have the same level of expertise.

## 2. Architecture Feedback / Improvements

### 2.1 Consider Changing Time Related Variable Declaration System

Time-related variables like _allowlistStartTime and endTime are based on block.timestamp. Therefore, declaring these variables requires adding block.timestamp and the expected time period for allowlisting. Instead, passing a time period for these variables as an argument and creating variables using block.timestamp and the time period within a function could simplify the process and make it more understandable.

### 2.2 Consider Decreasing The Number of Mappings

The NextGenCore contract currently includes 21 mappings, all linking collectionID's to other variables. Creating this many redundancy-prone storage variables isn't optimal. It's more efficient to create one variable connecting collectionID's to all other variables. Having this many redundant variables could lead to the creation of parallel data structures (tracking the same thing), which is considered risky. It's better to improve the contract's storage layout in this manner.

### 2.3 Consider Adding Checks for Edge Cases

Currently, nearly all admin-related functions can be called with arbitrary arguments. Given that some protocol admins may not be familiar with the space (use case where universities and companies deploy the codebase), it's advisable to limit expected inputs and check for edge cases. While not critical, it's possible to change some state variables to the same state, which can also be prevented with this approach.


### 2.4 Consider Following CEI Pattern and Usage of non-Reentrant Modifier

The protocol does not currently use the non-Reentrant modifier, and many functions do not follow the CEI pattern. During the codebase review, several reentrancy attacks were identified and reported. It's highly likely that the protocol has more reentrancy attack vectors that weren't detected in the limited review time, and more could emerge after the audit and mitigation processes are completed. It's crucial to understand that fewer attack vectors mean more secure code. With so many potential reentrancy attacks, it's risky to assume that all attacks have been identified in this contest.
###2.5 Consider Refunding Excess Ether Sent During Mint

During the minting process, the check verifies if the msg.value is greater than or equal to the NFT's price. However, any excess value is not refunded to the user. For improved user experience and satisfaction, it's recommended to implement a functionality that refunds any excess value sent.

# 3. Risks of Centralization

The codebase of the Nextgen Protocol is unfortunately highly centralized, possibly one of the most centralized in the space. All power is vested in the hands of admins, with various types of admins adding complexity to the execution process. Here are some of the risks associated with this:

- There's an emergencyWithdraw function that allows for a potential rug-pull scenario.
- Global admins hold extensive power. They can alter anything in the protocol, including Collection Data, mint tokens for anyone (via airdrop), and even freeze collections.
- Function admins, who are privileged in their respective functions, possess more power than the artists themselves. They can do everything a global admin can, but only within their specific functions.
- Artists have more control over their collections than necessary. If the collection is not frozen, they can alter token metadata, images, attributes, etc. This undermines one of the main benefits of a blockchain, which is immutability.

This level of centralization presents a significant risk and can compromise the integrity and security of the protocol.

### Time spent:
20 Hours



### Time spent:
20 hours
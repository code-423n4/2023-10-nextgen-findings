# Analysis - NextGen Contest

![NextGen-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2FneYKTfpiuGS.0&w=256&q=75)

## Description overview of The NextGen Contest

The NextGen Protocol aims to explore experimental directions in generative art as well as non-art uses of on-chain NFTs, It provides an extension of classic on-chain generative art models with additional features like phase-based releases, whitelist-based minting, and delegation, A key feature is the ability to pass arbitrary data to the contract to customize NFT outputs for individual addresses. This allows for experiments in personalized/dynamic art collections, different distribution methods, and collections with broader social applications beyond just art, It supports experimentation with different generative art techniques, concepts, aesthetics, technologies, and intellectual property, Personalization features enable experiments in different generative art models, unique trait distributions, and collections that are not strictly art.

**The key contracts of NextGen protocol for this Audit are**:

Auditing the key contracts of the NextGen Protocol is essential to ensure the security of the protocol. Focusing on them first will provide a solid foundation for understanding the protocol's operation and how it manages user assets and stability. Here's why it's important to audit these specific contracts:

- **NextGenCore.sol**: This is the Core contract that managing and minting NFTs. It includes features for creating collections, setting collection-specific data, minting tokens, handling airdrops, burning tokens, and various administrative functions.it features functionality that has not been extensively tested or audited before so the contract has many configurable features, nested structures, and external interactions that increase complexity and the potential for unintended bugs or vulnerabilities.

- **MinterContract.sol**: The Minter contract is used to mint an ERC721 token for a collection on the Core contract based on certain requirements that are set prior to the minting process. It implements complex functionality like different sales models, minting phases, royalty payments, etc. This complexity increases the risk of errors. In particular, errors in royalty calculation/distribution could result in lost funds for artists.

- **AuctionDemo.sol**: This contract implements a non-fungible token (NFT) auction system. Bidders can participate, place bids, and the highest bidder or admin can claim the NFT after the auction ends, so auditing this contract is important because mechanisms are needed to prevent bids from being front-run by other accounts, the auction rules should prevent newer smaller bids from overriding a previous highest bid, and the contract should refund any cancelled or invalid bids back to bidders.

As the audit of these contracts, I checked for potential security vulnerabilities, such as reentrancy, access control issues, and logic flaws. Additionally, thoroughly test the functions and roles defined in these contracts to make sure they behave as expected.

## System Overview

### Scope

- **NextGenCore.sol**: The NextGenCore contract is an implementation for managing non-fungible token (NFT) collections. It adheres to standards like ERC721Enumerable, Ownable, and ERC2981. This contract allows the creation, minting, and administration of NFT collections, featuring functionalities such as airdrops, artist signatures, and metadata management.

- **MinterContract.sol**: This contract defines a versatile NFT smart contract, incorporating features for minting, auctions, and royalties. It manages various collections with distinct phases, costs, and burn-to-mint functionalities. The contract supports external collection interactions, artist payments, and admin functions. Ownership and authorization controls are implemented through inheritance from the Ownable contract and various modifiers.

- **NextGenAdmins.sol**: The NextGen Admin Contract manages global and collection-specific administrators, enabling fine-grained control over contract functionalities. It includes features to register global admins, function-specific admins, batch function admins, and collection admins. The contract ensures that designated functions can only be executed by authorized administrators, with ownership privileges granted to the contract deployer. The contract provides retrieval functions for checking admin permissions, contributing to a robust access control system for contract administration.

- **RandomizerNXT.sol**: The contract facilitates a controlled and secure randomization process for token generation in the NextGen ecosystem.

- **RandomizerVRF.sol**: The NextGen Randomizer Contract VRF, utilizes Chainlink's VRF for secure randomness. It integrates with VRFCoordinatorV2Interface, Ownable, INextGenCore, and INextGenAdmins contracts. The contract facilitates random word generation, linking it to token IDs, and securely calculates token hashes for the NextGen ecosystem.

- **RandomizerRNG.sol**: The contract securely calculates token hashes, linking them to token IDs, and provides flexibility by allowing adjustments to the RNG cost. Additionally, an emergency withdrawal function ensures the safety of the contract's funds in unforeseen situations.

- **XRandoms.sol**: This contract offering random words from a list. It utilizes block-related data for randomness and provides functions to fetch random words, numbers, or specific words based on indices.

- **AuctionDemo.sol**: The Auction Demo Contract facilitates auctions for NFTs. It includes bid participation, tracking the highest bid and bidder, claiming the auctioned NFT, canceling individual bids, canceling all bids, and retrieving bid information. The contract ensures that only the highest bidder or admins can claim the auction. The bidding and claiming process involves transferring NFT ownership and handling refunds.

### Privileged Roles

Some privileged roles exercise powers over the controller of the contracts:

- **Admin**

```solidity
    modifier AdminRequired {
      require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
      _;
    }
```

- **Global-Admin**

```solidity
// retrieveGlobalAdmin
    modifier FunctionAdminRequired(bytes4 _selector) {
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
      _;
    }
```

- **Function-Admin**

```solidity
    modifier FunctionAdminRequired(bytes4 _selector) {
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
      _;
    }
```

- **Collection-Artist**

```solidity
    modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
      _;
    }
```

### Roles

The system defines different roles, including Admin, Function Admin, and Collection Admin.

1.  Admin: The primary role is to manage and control access to specific functions within the Core and Minter contracts. Admins can be classified into global admins, function admins, and collection admins.

    - Global Admin

      - Registration: The registerAdmin function allows the contract owner to register or deregister an Ethereum address as a global admin.
      - Status Check: The retrieveGlobalAdmin function checks the status of a given Ethereum address, determining whether it is registered as a global admin.

    - Function Admin:

      - Registration: The registerFunctionAdmin function, callable by global admins, registers or deregisters an Ethereum address as a function admin with permission to call specific functions.
      - Status Check: The retrieveFunctionAdmin function checks whether a given Ethereum address is registered as a function admin for a specific function.

    - Collection Admin:

      - Rgistration: The registerCollectionAdmin function, callable by global admins, registers or deregisters an Ethereum address as a collection admin for a specific collection.
      - Status Check: The retrieveCollectionAdmin function checks whether a given Ethereum address is registered as a collection admin for a specific collection.

## Approach Taken-in Evaluating The NextGen Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    The NextGen ecosystem comprises the NextGenCore managing NFT collections, MinterContract enabling versatile NFT functionalities, and NextGenAdmins for comprehensive admin controls. RandomizerNXT facilitates secure token randomization, while RandomizerVRF and RandomizerRNG ensure randomness with Chainlink VRF and flexible RNG costs. AuctionDemo orchestrates NFT auctions, integrating with MinterContract for bid handling and ownership transfers. These contracts collectively form a robust and feature-rich ecosystem for NFT creation, management, and auctioning.

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the NextGen Protocol.
    I started with the contract that is at the core of the system and from which other contracts interact or depend on. In this case.

    I start with the following contracts, which play crucial roles in the NextGen protocol system:

    **Main Contracts I Looked At**

                NextGenCore.sol
                MinterContract.sol
                AuctionDemo.sol
                Randomizer
                  -RandomizerVRF.sol
                  -RandomizerRNG.sol
                  -RandomizerNXT.sol

    I started my analysis by examining the NextGenCore.sol contract. Becuase this is the core NFT contract that implements features for creating collections, setting collection-specific data, minting tokens, handling airdrops, burning tokens.

    Then, I turned our attention to the MinterContract.sol. This contract provides complex functionality like different sales models, minting phases, royalty payments

    Then Dive into AuctionDemo.sol contract, This handles the auction mechanism for NextGen protocol.

    Then audit Randomizer contracts this defines the facilitates random word generation, linking it to token IDs.

2.  **Documentation Review**:

    Then went to Review [This Docs](https://seize-io.gitbook.io/nextgen/) for a more detailed and technical explanation of NextGen protocol.

3.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

4.  So in summary Found some Vulnerabilities I hope this will secure the NextGen Protocol

    - Duplicate or inconsistent indexes being generated if total supplies are adjusted unexpectedly

    - A lack of validation for recipient addresses, allowing tokens to potentially be sent to invalid or burnt addresses

    - Assumptions about the minter contract or network that are not verified could lead to unexpected behaviors

    - Metadata and array access logic does not include checks to prevent errors

    - No protections are in place to ensure testnet tokens can't be accidentally or maliciously minted on mainnet

    - External dependencies like randomizers are relied on without validating outputs

    - Collection finalization logic has no duplicate call prevention

- **Addressing these issues through strengthened input validation, error handling, and refactoring dependent relationships would help secure the contracts against exploitation or unintentional bugs.**

## Architecture Description and Diagram

Architecture of the key contracts that are part of the NextGen protocol:

![Diagram](https://i.im.ge/2023/11/11/yK47Iy.Screenshot-from-2023-11-11-11-26-19.png)

The NextGen protocol consists of key contracts: Core, Minter, Admin, and Randomizer, collectively forming a robust NFT ecosystem. The Core contract handles ERC721 token minting and integrates with other contracts for enhanced functionality. Minter oversees the minting process with details on drops, phases, artists, and sales models. Admin manages global and function-specific admins for controlled contract access. Randomizer generates random hashes for token creation, with options for Chainlink VRF, ARRNG, or custom implementations. This architecture ensures a flexible and scalable NFT creation and management system.

Overall, the NextGen architecture is well designed and thought out to provide a flexible, customizable and scalable solution for minting and managing NFT collections and sales

### Some potential areas for improvement and Architecture feedback

1. As the system scales, performance and gas costs of on-chain metadata queries could become an issue. Offloading non-critical metadata to IPFS may help.

2. Formal verification of key functions like minting would provide stronger guarantees on functionality and security using a tool like Certora.

3. Support for royalties on secondary sales directly on the smart contract. This would allow creators to easily collect royalties without relying on third party marketplaces.

4. Introduce token governance features - allow community to vote on collection parameters, proposal of new features, emergency upgrades etc.

5. Support batch minting and actions like purchasing multiple tokens in one transaction for better UX and lower fees.

6. Research integrating layer 2 scaling like zkRollups to scale metadata/royalty transfers beyond current Ethereum limits.

7. Explore creative crowdfunding/accelerator programs to support early stage generative art projects utilizing the platform.

8. Extensive documentation & tutorials - detailed guides, cookbooks and workshops to educate people on how to leverage the full power of the platform. Lower the barrier of entry.

## Codebase Quality

Overall, I consider the quality of the NextGen protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Code Maintainability and Reliability** | The NextGen Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                           |
| **Code Comments**                        | During the audit of the NextGen contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                     |
| **Documentation**                        | The documentation of the NextGen project is quite comprehensive and detailed, providing a solid overview of how NextGen protocol is structured and how its various aspects function. However, we have noticed that there is room for additional details, such as diagrams, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 100% this is Excellent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. It inherits from multiple components, adhering for-example The NextGenCore contract follows a modular structure with clear separation of concerns into Core contract logic, inherited ERC standards, and imported supporting contracts.Functions are grouped thematically and formatted uniformly with visibility, modifiers, and error handling.Configurable collections and tokens are modeled with nested structured data mapped to indexes for gas efficiency.                                                                                                                                                         |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the NextGen protocol. These risks encompass concentration risk in NextGenCore, MinterContract, AuctionDemo risk and more, third-party dependency risk, and centralization risks arising from the existence of an “owner” role in specific contracts. However, the documentation lacks clarity on whether this address represents an externally owned account (EOA) or a contract, warranting the need for clarification. Additionally, the absence of fuzzing and invariant tests could also pose risks to the protocol’s security.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **No having, fuzzing and invariant tests could open the door to future vulnerabilities**.

2. The NextGenCore.sol contract's design could be enhanced to prevent redundant data if invalid collections are created by an Admin. Currently, the `newCollectionIndex` increments unconditionally when createCollection function is called, even if erroneous data is provided. This means administrators cannot delete improper entries, and must create new collections instead. Over time, this approach may result in stale or obsolete data accumulating within the collectionInfo mapping if no logic is added to cleanup or overwrite failed creation attempts. Considering alternatives to increment indexes only on successful validation could help avert persisting of invalid records.

3. Minting flows, royalty payments fully controlled by contracts without oversight. Rigs to get stuck if bugs exist.

4. The MinterContract.sol involves a delegation system where certain addresses can delegate their minting rights to others. Depending on the implementation and usage, this could introduce risks related to centralization if a small number of addresses control a significant portion of the minting process.

5. The airDropTokens function in the MinterContract contract allows for the airdropping of tokens to multiple addresses. Depending on the distribution mechanism and the entities controlling the airdrop, this could introduce systemic risks.

6. The addresses of the external contracts are hardcoded in the constructor. If these addresses change or if the contract is deployed on a different network, it would require manual updates to these addresses, potentially leading to errors or vulnerabilities.

7. RandomizerNXT.sol contract reliance on blockhash to generate randomness exposes risks if blockchain reorgs.

### Centralization Risks:

1.  The contract designates certain functions that can only be called by global or function admins. This centralization of administrative control could pose a risk if the admin accounts are compromised or act maliciously.

2.  The ability to freeze a collection in the NextGenCore contract through the freezeCollection function is centralized and can be controlled by administrators. This can be a risk if a single entity has too much power.

3.  Off-chain data like collection info is defined and stored on-chain rather than IPFS/Arweave, increasing centralization.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the NextGen protocol.**

### Summary

Heavy reliance on centralized admins accounts. Many critical functions can only be called by specific centralized accounts.This level of control invested in a small set of accounts reduces the decentralization of the system.

## Gas-Optmization Summary

- Avoid unnecessary checks like contract existence checks before external calls to reduce gas.
- Be mindful of string and data sizes to stay within EVM limits and reduce truncated errors.
- Initialize storage variables directly instead of incrementing from zero to reduce gas costs.
- Access storage directly instead of copying to memory for further gas savings.
- Consider trade-offs between modifiers and internal functions like bytecode size.
- Use signatures over merkle proofs for allowlists and airdrops for better scaling.
- ERC1155 tokens have better storage efficiency than ERC721 in many cases.
- Explore gas-optimized alternatives to OpenZeppelin for certain functions.
- Use inline assembly for reverts instead of Solidity for lower gas.
- Reuse memory between external calls to avoid memory expansion.
- Named returns generate better bytecode than anonymous returns.
- Loops like do-while can be more efficient than for loops in some cases.
- Expressions can short-circuit to skip unnecessary evaluations.
- Avoid unnecessary public variables to reduce contract size and costs.
- Caching calldata reads can optimize repeated lookups.
- Remove empty blocks or add emissions to avoid wastage.
- Access globals directly instead of caching local copies.
- Inline single-use modifiers to remove bytecode overhead.
- Pre-increment/decrement operators are more efficient than addition/subtraction.
- Use bytes.concat() over abi.encodePacked() since 0.8.4.
- Cache external calls outside loops to avoid re-calling on each iteration.

## Conclusion

In general, The NextGen protocol presents a well-designed architecture for managing generative NFT collections with its set of core contracts, we believe the team has done a good job regarding the code. However, the analysis revealed some security and decentralization risks that warrant attention. Chief among these is over-reliance on centralized admin accounts for critical functions. Additionally, it is recommended to improve the documentation and comments in the code to enhance
understanding and collaboration among developers and auditors. It is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.


### Time spent:
35 hours
# NextGen - Advanced smart contracts for launching generative art projects on Ethereum.
![Banner](https://github.com/0xWeb3boy/photo/assets/113019033/11ec09fb-781a-471d-abca-d1a784292d64)

# Teaser Video 

https://github.com/0xWeb3boy/photo/assets/113019033/d321f91b-f889-47ea-bf87-7762cdbe66df

## Table of Contents

- 1 . Executive Summary
- 2 . Code Audit Approach
  - 2.1 Audit Documentation and Scope
  - 2.2 Code review
  - 2.3 Threat Modelling
  - 2.4 Exploitation and Proofs of Concept
  - 2.5 Report Issues
- 3 . Architecture overview
  - 3.1 Overview of NextGen  
  - 3.2  Key Contracts Introduced
  - 3.3 Scope of the Audit
- 4 . Implementation Notes
  - 4.1 General Impressions
  - 4.2 Composition over Inheritance
  - 4.3 Centralization risks
  - 4.4 Comments
  - 4.5 Solidity Versions
- 5 . Conclusion

## 1. Executive Summary

In focusing on the ongoing audit contest NextGen , my analysis starts by delineating the code audit methodology applied to the contracts within the defined scope. Subsequently, I provide insights into the architectural aspects, offering my perspective. Finally, I offer observations pertaining to the code implementation.


I want to emphasize that unless expressly specified, any potential architectural risks or implementation concerns discussed in this document should not be construed as vulnerabilities or suggestions to modify the architecture or code based solely on this analysis. As an auditor, I recognize the necessity of a comprehensive evaluation of design choices in intricate projects, considering risks as only one component of a larger evaluative process. It's essential to acknowledge that the project team may have already evaluated these risks and established the most appropriate approach to mitigate or coexist with them.

## 2. Code Audit Approach

Time spent: 18 hours

### 2.1 Audit Documentation and Scope
Commencing the analysis, the first phase entailed a thorough review of the https://github.com/code-423n4/2023-10-nextgen to fully grasp the fundamental concepts and limitations of the audit. This initial step was crucial in guiding the prioritization of my audit efforts.

### 2.2 Code review


Initiating the code review, the starting point involved gaining a comprehensive understanding of 

## MinterContract.sol 
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol
## NextGenCore.sol
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol
## RandomizerVRF.sol
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol

all these are a pivotal components responsible for implementing NextGen architecture. This understanding of the core pattern significantly facilitated the comprehension of the protocol contracts and their interconnections. During this phase, meticulous documentation of observations and the formulation of pertinent questions regarding potential exploits were undertaken, striking a balance between depth and breadth of analysis.

### 2.3 Threat Modelling

The initial step involved crafting precise assumptions that, if breached, could present notable security risks to the system. This approach serves to guide the identification of optimal exploitation strategies. Although not an exhaustive threat modeling exercise, it closely aligns with the essence of such an analysis.

### 2.4 Exploitation and Proofs of Concept

Progressing from this juncture, the primary methodology took the form of a cyclic process, conditionally encompassing steps 2.2, 2.3, and 2.4. This involved iterative attempts at exploitation and the subsequent creation of proofs of concept, occasionally aided by available documentation or the helpful community on Discord. The key focus during this phase was to challenge fundamental assumptions, generate novel ones in the process, and refine the approach by utilizing coded proofs of concept to hasten the development of successful exploits.

### 2.5 Report Issues

While this particular stage might initially appear straightforward, it harbors subtleties worth considering. Hastily reporting vulnerabilities and subsequently overlooking them is not a prudent course of action. The optimal approach to augment the value delivered to sponsors (and ideally, to auditors as well) entails thoroughly documenting the potential gains from exploiting each vulnerability. This comprehensive assessment aids in the determination of whether these exploits could be strategically amalgamated to generate a more substantial impact on the system's security. It's important to recognize that seemingly minor and moderate issues, when skillfully leveraged, can compound into a critical vulnerability. This assessment must be weighed against the risks that users might encounter. Within the realm of Code4rena audit contests, a heightened level of caution and an expedited reporting channel are accorded to zero-day vulnerabilities or highly sensitive bugs impacting deployed contracts.

## 3. Architecture overview

![Arch](https://github.com/0xWeb3boy/photo/assets/113019033/f2542390-0299-497c-9c19-e6dd6b840536)



### 3.1 Overview of NextGen  

NextGen is a sophisticated system of smart contracts that offers the ability to generate numerous ERC721 collections using a single smart contract address. The architecture of NextGen comprises four primary smart contracts: Core, Minter, Admin, and Randomizer. The Core contract acts as the central hub, connecting and enabling interaction between the other contracts, thereby delivering adaptable and scalable functionalities.


> NextGen represents a series of contracts aimed at exploring new frontiers in generative art and the utilization of entirely on-chain NFTs for purposes beyond art.
> Conceptually, NextGen functions as an on-chain generative contract with expanded capabilities. It amalgamates principles from The Memes, employing phase-based, allowlist-based, and delegation-based minting philosophies.
> It incorporates the unique feature of enabling specific addresses to customize outputs by transmitting custom data to the contract.
> The framework boasts a versatile spectrum of minting models, each assignable to distinct phases, offering a broad range of creative possibilities.
> Introduction to foundational concepts is provided, allowing those familiar with the terms to navigate directly to sections relevant to their interests.


### 3.2 Key Contracts Introduced:

#### MinterContract.sol

Within the NextGen ecosystem, the Minter contract operates as the pivotal mechanism responsible for the creation of ERC721 tokens tailored to a specific collection overseen by the Core contract. It orchestrates this minting process in accordance with predetermined criteria established prior to initiating the minting protocol. This contract serves as the repository of crucial details pertinent to an impending token release, encompassing an array of essential information.

Central to its function, the Minter contract houses a comprehensive repository encompassing various intricacies of an upcoming token drop. These encompass, but are not limited to, the delineation of starting and concluding times for different phases, intricate Merkle roots, the sales model adopted, allocation of funds, and the primary as well as secondary addresses associated with the contributing artists. Essentially, it stands as the hub storing the blueprint that dictates the temporal, structural, and logistical framework for the impending token release within the ecosystem.

#### NextGenCore.sol

At the heart of the infrastructure lies the Core contract, a pivotal locus where the ERC721 tokens come into existence, housing not only the fundamental functionalities outlined within the ERC721 standard but also extending its capabilities through a suite of additional setter and getter functions. This integral contract serves as the custodian of crucial data associated with a given collection within the ecosystem.

Immersed in its repository are multifaceted details that define the essence of a collection, ranging from elemental aspects like the collection's name, the identity of the contributing artist, the library utilized, the underlying script, and the overarching quantitative aspect encapsulated by the total supply of the collection. These nuanced details collectively shape the identity and substance of the collection being facilitated within the Core contract.

Moreover, the Core contract doesn't operate in isolation; instead, it intricately intertwines with the suite of other NextGen contracts, thereby engendering a dynamic and adaptable functionality. This interconnection serves as the linchpin for a flexible, modifiable, and scalable ecosystem, ensuring a coherent and seamless integration of various features and functionalities across the NextGen framework.

#### Randomizer Architecture

The pivotal role of the Randomizer contract within the NextGen framework is the facilitation of randomness encapsulated in a unique hash for every token minted within the system. This essential process transpires during the minting phase, where the Randomizer contract assumes the responsibility of generating and assigning an individualized, unpredictable hash to each token.

Subsequently, this newly minted and distinct hash isn't an endpoint but rather a crucial transitional phase. Once formulated, the hash embarks on a journey, transitioning to its designated abode within the Core contract. Within the Core's repository, these generated hashes find their dwelling, poised and ready to be harnessed in the creation of the generative art tokens.

The significance lies in the orchestrated transfer of these hashes to the Core, positioning them as foundational components for the eventual genesis of generative art tokens. This process ensures that the randomness injected by the Randomizer becomes an integral element in the generative art token creation process, thereby contributing to the unique and unpredictable nature of each resultant token.

### 3.3 Scope of the Audit

| Contract |SLOC  |Purpose |
|:--|:----------------|:------|
|[NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol) |366 |Core is the contract where the ERC721 tokens are minted and includes all the core functions of the ERC721 standard as well as additional setter & getter functions. |
|[MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol) |475 | The Minter contract is used to mint an ERC721 token for a collection on the Core contract based on certain requirements that are set prior to the minting process.  |
|[NextGenAdmins.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol) |61 |The Admin contract is responsible for adding or removing global or function-based admins who are allowed to call certain functions in both the Core and Minter contracts. |
|[AuctionDemo.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol) |114 | The auctionDemo smart contract holds the current auctions after the mintAndAuction functionality is called. Users can bid on a token and the highest bidder can claim the token after an auction finishes. |
|[RandomizerNXT.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol) |51 |The RandomizerNXT contract is responsible for generating a random hash for each token during the minting process using the NextGen's proposed approach.|
|[RandomizerVRF.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol) |87 | The RandomizerVRF contract is responsible for generating a random hash for each token during the minting process using the Chainlink's VRF service. |
|[RandomizerRNG.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol) |72 | The RandomizerRNG contract is responsible for generating a random hash for each token during the minting process using the ARRng.io service. |
|[XRandoms.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol) |39 | 	The randomPool smart contract is used by the RandomizerNXT contract, once it's called from the RandomizerNXT smart contract it returns a random word from the current word pool as well as a random number back to the RandomizerNXT smart contract which uses those values to generate a random hash.|




## 4. Implementation Notes

During the course of the audit, several noteworthy implementation details were identified, and among these, a significant subset holds potential value for the ongoing analysis.

### 4.1 General Impressions

# Overview of NextGen's Architecture

NextGen's Goals:

The quintessential paradigm within a NextGen collection adheres to a fully on-chain model, reminiscent of renowned reference generative art collections such as Autoglyphs and Art Blocks. At its core, this on-chain essence epitomizes a shared fundamental trait, encapsulating crucial attributes that ensure the collection's self-contained nature within the contract framework.

An elemental aspect defining this on-chain constitution involves the storage of pivotal components—namely, the script, library link, and seed—integral in generating and rendering images directly from the contract data. This storage paradigm ensures that any image rendered in the future is derived directly from the stored data within the contract, embracing a self-sufficiency that characterizes the entire collection.

The Randomizer contract assumes a pivotal role in this on-chain orchestration by generating a unique hash for each minted token. These hashes, stored on the Core contract, serve as the foundational elements utilized by the generative script to craft the final output, imbuing each token with an individualized and inherently unpredictable nature.

Furthermore, NextGen bolsters this on-chain framework by introducing three distinct features to enrich the on-chain repository:

The modification of the tokenURI to support both off-chain and on-chain metadata, empowering admins to alter how the metadata for a collection is accessed when the tokenURI function is invoked.
The capability to support on-chain metadata, whereupon enabling this feature, NextGen commences the production of an on-chain .json object for each token. This object encapsulates vital parameters such as token name, description, image, attributes, and animation_url, facilitating a comprehensive and self-contained token identity within the on-chain environment until the collection is locked.
Notably, artists can affix their signature on-chain, linking their Ethereum address or ENS to a specific collection. This enriches the provenance by ensuring the on-chain presence of an artist's signature alongside the collection.
While many generative collections conventionally point to an externally hosted image for marketplaces and other services, NextGen diverges by integrating a similar approach for visualization during minting. However, it distinguishes itself by introducing a novel feature allowing post-minting administrators to update the off-chain metadata URI. This update facilitates the inclusion of a rendered image on decentralized storage platforms like IPFS or Arweave, eliminating any reliance on the creator's infrastructure after minting, thereby ensuring a robust and self-sufficient framework independent of external dependencies.

### 4.2 Inheritance over Composition

NextGen is using Inheritance over composition, Inheritance in software design relies on a hierarchical structure where classes or objects inherit properties and behavior from a parent class or base object. While inheritance offers simplicity and a hierarchical organization, it often lacks the flexibility and adaptability found in composition. Inheritance can lead to a rigid code structure where changes in the base class may impact all derived classes, potentially creating a tightly coupled system that is challenging to maintain and test.

Moreover, inheritance can present challenges like the diamond inheritance problem, where multiple classes inherit from a common base, leading to ambiguity and complexities in method resolution. While inheritance can promote code reuse to some extent, it tends to bind components more closely and may not allow for the dynamic assembly of functionality that composition enables.

Overall, inheritance can provide a straightforward and organized structure but may lack the versatility, scalability, and modularity offered by composition. The choice between composition and inheritance often depends on the specific requirements of a software system and the trade-offs between simplicity and adaptability.

### 4.3 Centralization risks

Some privileged roles exercise powers over the Controller contract:

- **Certain functions can only be called by an admin or the artist**

```solidity
  modifier ArtistOrAdminRequired(uint256 _collectionID, bytes4 _selector) {
      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
      _;
```

- **Certain functions can only be called by a global or function admin**

```solidity
 modifier FunctionAdminRequired(bytes4 _selector) {
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
      _;
    }
```

- **Certain functions can only be called by a collection, global or function admin**

```solidity
 modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
      _;
    }
```

### 4.4 Comments

#### Importance of Comments for Clarity:

Comments serve a pivotal role in enhancing the understandability of the codebase. While the code is generally clean and logically structured, judiciously placed comments can provide valuable insights into the functionalities and intentions behind the code. They contribute to a better comprehension of the code's purpose, especially for auditors and developers involved in the analysis.

#### Strategic Comment Placement:

The codebase would greatly benefit from an increased presence of comments. Strategic placement of comments within the essential contracts can significantly aid auditors in comprehending the implementation details. These comments should elucidate the logic, processes, and methodologies employed, promoting a seamless audit experience.

#### Facilitating Audits and Code Readability:

Comprehensive comments in the all of the contracts can expedite the auditing process by allowing auditors to swiftly grasp the intended functionality of the code. This, in turn, enhances the readability of the functions and methods, making it easier for auditors to identify any inconsistencies or deviations between the documented intentions and the actual code implementation.

#### Detecting Discrepancies in Code Intentions:

An important aspect of code review is discerning any mismatches between the documented intentions in the comments and the actual code implementation. Comments that accurately reflect the code's purpose are crucial for auditors, as discrepancies between the two can be indicators of potential vulnerabilities or errors. The act of aligning comments with the true code behavior is fundamental to ensuring the reliability and security of the smart contract.


### 4.5 Solidity Versions

Though there are valid arguments both in support of and against adopting the latest Solidity version, I find that this discussion bears little significance for the current state of the project. Without a doubt, choosing the most up-to-date version is a far superior decision when compared to the potential risks associated with outdated versions.


## 5. Conclusion

### Positive Audit Experience:

The process of auditing this codebase and evaluating its architectural choices has been thoroughly enjoyable and enriching. Navigating through the intricacies and nuances of the project has been enlightening, presenting an opportunity to delve into the complexities of the system.

### Strategic Simplifications for Complexity Management:

Inherent complexity is a characteristic of many systems, especially those in the realm of blockchain and smart contracts. The strategic introduction of simplifications within this project has proven to be a valuable approach. These simplifications are well-thought-out and strategically implemented, demonstrating an understanding of how to manage complexity effectively.

### Achieving a Harmonious Balance:

A notable achievement of this project is striking a harmonious balance between the imperative for simplicity and the challenge of managing inherent complexity. This equilibrium is crucial in ensuring that the codebase remains comprehensible, maintainable, and scalable, even as the system becomes more intricate.

### Importance of Methodology Overview:

The overview provided regarding the methodology employed during the audit of the contracts within the defined scope is invaluable. It offers a clear and structured insight into the analytical approach undertaken, shedding light on the depth and rigor of the evaluation process.

### Relevance for Project Team and Stakeholders:

The insights presented are not only beneficial for the project team but also extend to any party with an interest in analyzing this codebase. The detailed observations, considerations, and recommendations have the potential to guide and inform decision-making, contributing to the project's overall improvement and security.


### Time spent:
18 hours

### Time spent:
18 hours
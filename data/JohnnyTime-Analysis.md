# NextGen - JohnnyTime’s Analysis Report

![NextGen](https://i.imgur.com/Q5x3h9l.png)

# Description

NextGen is a series of smart contracts designed to explore experimental aspects of **generative art** and expand the use of 100% on-chain NFTs **beyond art-related applications**. 

NextGen combines the features of a traditional on-chain generative contract with the minting approach of The Memes, featuring **phase-based**, **allowlist-based**, and **delegation-based** minting philosophies. 

Additionally, it allows for the customization of outputs by passing specific data to the contract for particular addresses. It offers various minting models that can be associated with different phases. NextGen comprises four primary smart contracts: **Core**, **Minter**, **Admin**, and **Randomizer**, collectively providing adaptable and scalable functionality for creating multiple ERC721 collections within a single smart contract address.

# Approach

Throughout the analysis process, my primary goal was to achieve a comprehensive understanding of the codebase, with the ultimate goal of creating well-informed recommendations to enhance its operational efficacy.

**The examined contracts in-scope were:**

```
1. NextGenCore.sol
2. MinterContract.sol
3. NextGenAdmin.sol
4. RandomizerRNG.sol
5. RandomizerVRF.sol
6. RandomizerNXT.sol
```

I initiated the analysis by first covering the **Core** contract, followed by an examination of the **Minter** contracts, which constitute the primary logic within the system. Subsequently, I proceeded to assess the remaining contracts.

This approach allowed me to work with the contract housing the most critical logic, often referred to as the "Father Contract", to gain a thorough understanding of the system's primary logic and operational flow. From there, I delved into the other smaller "helper" contracts.

I invested more than 60 hours during the 14-day contest to conduct a comprehensive audit of the codebase. My objective is to add value to the project team with an exhaustive report that affords them a broader perspective, enhancing the value of my research by fortifying the security, user-friendliness, and efficiency of the protocol.

# Architecture

![NextGen-Architecture](https://i.imgur.com/Agc2UcC.png)

**NextGenCore.sol:** The Core contract is the central hub for minting ERC721 tokens. It encompasses all the fundamental functions of the ERC721 standard and includes extra setter and getter functions. This contract stores essential collection data, including the collection's name, artist's name, library, script, and total supply. Moreover, it collaborates with other NextGen contracts to deliver adaptable and scalable functionality.

**NextGenMinter.sol:** The Minter contract is responsible for minting ERC721 tokens for a collection hosted on the Core contract, adhering to predefined requirements. This contract manages crucial information about upcoming drops, such as phase start and end times, Merkle roots, sales models, funds, and the primary and secondary addresses of artists.

**Randomizer Contracts:** The Randomizer contracts play a pivotal role in generating a random hash for each token during the minting process. Once the hash is generated, it is transmitted to the Core contract, where it is stored for use in generating generative art tokens. 

NextGen offers three distinct Randomizer contracts: one that utilizes **Chainlink VRF (RandomizerVRF.sol)**, another that uses **ARRNG (RandomizerRNG.sol)**, and a **custom-made implementation (RandomizerNXT.sol)** of the Randomizer contract.

**NextGenAdmin.sol:** The Admin contract handles the addition or removal of global or function-specific administrators who have authorization to execute certain functions within both the Core and Minter contracts.

# Codebase Quality and Issues

In general, I find the NextGen codebase to be of moderate quality. This assessment is based on various factors that I will now outline:

## Code Comments

The code comments were not helpful and made it tough to understand what the code does.

Function comments, like this one:

```solidity
// mint and auction
    
function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
    require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
    uint256 collectionTokenMintIndex;
```

Aren't very useful, especially when dealing with long and confusing functions where the parameter names aren't always easy to grasp.

It appears that the developers did create comprehensive Natspec comments for the functions, but oddly, they included them in the [function documentation on the website](https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/minter), not within the actual codebase:

![Source: NextGen docs](https://i.imgur.com/WmgbqEl.png)

This required me to manually go through each function on the website and copy the Natspec documentation into my local codebase in VSCode, significantly extending the auditing process's duration and reducing its efficiency.

Furthermore, in some instances, the code comments are inaccurate and fail to align with the current data or functionality. An example from the `AuctionDemo.sol` contract:

![wrong-code-comments](https://i.imgur.com/Ts6MVTD.png)

### ✅ Recommendations

I strongly advise incorporating the detailed Natspec comments directly into the codebase. This will benefit developers, auditors, and users by providing clear insights into each function's purpose. Additionally, blockchain explorers often rely on these Natspec comments, making it easier for users to interact with smart contracts via blockchain explorers.

In addition to that, make sure all the code comments are correct.

## Redundant Code

The codebase contains numerous instances of redundant code, where the same code is re-implemented and repeated multiple times. This is generally regarded as poor coding practice for several reasons:

- Firstly, it leads to lengthy and complex functions, making it challenging for security researchers to work with and comprehend.
- Additionally, it introduces the potential for errors and bugs, as inconsistencies may arise when the same logic is implemented multiple times in different functions.

Here are some examples of redundant code that can benefit from improvement:

1. The identical `updateAdminsContract` function is duplicated in all other smart contracts within the system. A more efficient approach would involve making the `NextGenAdmin.sol` contract upgradable.
2. Within the Minter contract, numerous Minting functions share similar logic with minor variations. I recommend extracting this common logic into an external function that can be utilized across all mint functions. For instance, the redundant delegation checks present in both the `mint` and `burnOrSwapExternalToMint` functions could be consolidated into an external function “checkDelegation” for improved code organization and efficiency:

```solidity
if (msg.sender != ownerOfToken) {
      bool isAllowedToMint;
      isAllowedToMint =
          dmc.retrieveGlobalStatusOfDelegation(
              ownerOfToken,
              0x8888888888888888888888888888888888888888,
              msg.sender,
              1
          ) ||
          dmc.retrieveGlobalStatusOfDelegation(
              ownerOfToken,
              0x8888888888888888888888888888888888888888,
              msg.sender,
              2
          );
      if (isAllowedToMint == false) {
          isAllowedToMint =
              dmc.retrieveGlobalStatusOfDelegation(
                  ownerOfToken,
                  _erc721Collection,
                  msg.sender,
                  1
              ) ||
              dmc.retrieveGlobalStatusOfDelegation(
                  ownerOfToken,
                  _erc721Collection,
                  msg.sender,
                  2
              );
      }
      require(isAllowedToMint == true, "No delegation");
  }
```

Additionally, the check for one minting per period, which is found in both the `mint` and `mintAndAuction` functions, can be abstracted into an external function named "enforceOnePerPeriod" for more streamlined code management:

```solidity
uint timeOfLastMint;
// check 1 per period
if (lastMintDate[_collectionID] == 0) {
    // for public sale set the allowlist the same time as publicsale
    // @audit-info first time 
    timeOfLastMint =
        collectionPhases[_collectionID].allowlistStartTime -
        collectionPhases[_collectionID].timePeriod;
} else {
    timeOfLastMint = lastMintDate[_collectionID];
}
// uint calculates if period has passed in order to allow minting
uint tDiff = (block.timestamp - timeOfLastMint) /
    collectionPhases[_collectionID].timePeriod;
// users are able to mint after a day passes
require(tDiff >= 1, "1 mint/period");
```

1. A lot of require statement both in the Minter and Core contracts are redundant and can be extracted to modifiers or functions, here are some:

```solidity
// Exist in many functions in the Core contract
require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");
```

```jsx
// Exist in 2 functions in the Minter contract
require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
```

```solidity
// Exist in 3 functions in the Minter contract
require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
```

```solidity
// Exist in 5 functions in the Minter contract
require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply" );
```

1. The use of "if", "elseif", and "else" blocks can be confusing and create potential for errors. Take, for instance, the `setCollectionData()` function, which could be made more straightforward. I suggest to simplify such implementations, for instance, the current implementation appears as follows: 

```solidity
function setCollectionData(
    uint256 _collectionID,
    address _collectionArtistAddress,
    uint256 _maxCollectionPurchases,
    uint256 _collectionTotalSupply,
    uint _setFinalSupplyTimeAfterMint
)
    public
    CollectionAdminRequired(_collectionID, this.setCollectionData.selector)
{
    require(
        (isCollectionCreated[_collectionID] == true) &&
            (collectionFreeze[_collectionID] == false) &&
            (_collectionTotalSupply <= 10000000000),
        "err/freezed"
    );
    if (
        collectionAdditionalData[_collectionID].collectionTotalSupply == 0
    ) {
        collectionAdditionalData[_collectionID]
            .collectionArtistAddress = _collectionArtistAddress;
        collectionAdditionalData[_collectionID]
            .maxCollectionPurchases = _maxCollectionPurchases;
        collectionAdditionalData[_collectionID]
            .collectionCirculationSupply = 0;
        collectionAdditionalData[_collectionID]
            .collectionTotalSupply = _collectionTotalSupply;
        collectionAdditionalData[_collectionID]
            .setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;

        collectionAdditionalData[_collectionID]
            .reservedMinTokensIndex = (_collectionID * 10000000000);

        collectionAdditionalData[_collectionID].reservedMaxTokensIndex =
            (_collectionID * 10000000000) + _collectionTotalSupply - 1;
            
        wereDataAdded[_collectionID] = true;
    } else if (artistSigned[_collectionID] == false) {
        collectionAdditionalData[_collectionID]
            .collectionArtistAddress = _collectionArtistAddress;
        collectionAdditionalData[_collectionID]
            .maxCollectionPurchases = _maxCollectionPurchases;
        collectionAdditionalData[_collectionID]
            .setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
    } else {
        collectionAdditionalData[_collectionID]
            .maxCollectionPurchases = _maxCollectionPurchases;
        collectionAdditionalData[_collectionID]
            .setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
    }
}
```

I recommend changing it to a simpler implementation, such as:

```solidity
function setCollectionData(
    uint256 _collectionID,
    address _collectionArtistAddress,
    uint256 _maxCollectionPurchases,
    uint256 _collectionTotalSupply,
    uint _setFinalSupplyTimeAfterMint
)
    public
    CollectionAdminRequired(_collectionID, this.setCollectionData.selector)
{
    require(
        (isCollectionCreated[_collectionID] == true) &&
            (collectionFreeze[_collectionID] == false) &&
            (_collectionTotalSupply <= 10000000000),
        "err/freezed"
    );

		// Brand new collection
    if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
      collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
			collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
      collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
      collectionAdditionalData[_collectionID].reservedMaxTokensIndex =(_collectionID * 10000000000) + _collectionTotalSupply - 1;
      wereDataAdded[_collectionID] = true;
		}
	  
		// Artist didn't sign yet
		if (artistSigned[_collectionID] == false) {
        collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
		}
		
		// These attributes will be updated anyhow
    collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
}
```

### ✅ Recommendations

1. Remove the `updateAdminsContract()` function from all contracts and transition the `NextGenAdmins.sol` contract to an upgradable upgradable contract.
2. Simplify the functions within the contracts, particularly those related to minting, by abstracting common and repetitive logic into external functions. The redundant code I've highlighted serves as a starting point, but the goal is to systematically review all contracts and generalize and extract any redundant functionalities into functions or modifiers. This will enhance code efficiency and maintainability.

## Usage of Hardcoded Values Instead of Constants

Utilizing **constants** instead of **hardcoded values** in Solidity smart contracts holds paramount importance for several reasons:

1. First and foremost, it significantly **enhances the clarity** and comprehensibility of the code. Constants provide meaningful names to values, which can be easily understood by developers and auditors alike. This naming convention imparts a level of self-documentation to the code, reducing the need for extensive comments or external documentation. 
2. Moreover, constants help prevent mistakes by centralizing the values, making it less likely for errors to occur due to discrepancies between multiple occurrences of the same value. When updates or modifications are needed, changing a constant value in one place automatically applies the update across the entire contract, enhancing code maintainability and reducing the risk of introducing bugs during the modification process.

During my in-depth analysis of the code, I uncovered several instances where redundant hardcoded values were employed instead of utilizing constant variables. This practice not only hampers the code's clarity and readability but also poses a risk of introducing errors or inconsistencies. **Here are some examples:**

1. Within the minter contract, there are delegation checks that involve the NFTDelegate smart contract. These checks include a parameter of the collection contract address, and a `_useCase`. At first glance, the meaning of these values wasn't immediately apparent, necessitating clarification from the [sponsor in our Discord chat](https://discord.com/channels/810916927919620096/1166760088963383336/1169571950457266186):
    
    ![Source: Code4Rena Discord Server, NextGen contest channel](https://i.imgur.com/fIuG0bH.png)
    
    ![Source: Code4Rena Discord Server, NextGen contest channel](https://i.imgur.com/7n3rqSm.png)
    
    Following the sponsor's explanation, it became clear that when the address is set to `0x000...`, it denotes the absence of access, while `0x8888...` indicates access to all collections. To enhance clarity and avoid such confusion, it's advisable to replace these values with constant variables bearing more comprehensible names, like `NO_DELEGATION_ACCESS` for `0x000...` and `ALL_COLLECTIONS_DELEGATION_ACCESS` for `0x8888…`:
    

```solidity
isAllowedToMint =
dmc.retrieveGlobalStatusOfDelegation(
    _delegator,
    0x8888888888888888888888888888888888888888,
    msg.sender,
    1
) ||
dmc.retrieveGlobalStatusOfDelegation(
    _delegator,
    0x8888888888888888888888888888888888888888,
    msg.sender,
    2
);
```

1. Another example can be found in the core contract, where the number `10000000000` recurs throughout the code. This number serves as the "gap" between token IDs among different collections, with each collection reserving a range of `10000000000` IDs:
    
    ```solidity
    collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
    ```
    
    Last but not least are the values `999` and `1000` within the core contract. These values play a crucial role in the `updateCollectionInfo` function, guiding how collection attributes are updated:
    
    ```solidity
    if (_index == 1000) {
        collectionInfo[_collectionID].collectionName = _newCollectionName;
        collectionInfo[_collectionID].collectionArtist = _newCollectionArtist;
        collectionInfo[_collectionID].collectionDescription = _newCollectionDescription;
        collectionInfo[_collectionID].collectionWebsite = _newCollectionWebsite;
        collectionInfo[_collectionID].collectionLicense = _newCollectionLicense;
        collectionInfo[_collectionID].collectionLibrary = _newCollectionLibrary;
        collectionInfo[_collectionID].collectionScript = _newCollectionScript;
    } else if (_index == 999) {
        collectionInfo[_collectionID].collectionBaseURI = _newCollectionBaseURI;
    } else {
        collectionInfo[_collectionID].collectionScript[_index] = _newCollectionScript[0];
    }
    ```
    

### ✅ Recommendations

1. Create of a new Solidity file `Constants.sol`. In this file include all the constants of the protocol.
2. Systematically replace all hardcoded values within the codebase with constants that will be stored within the `Constants.sol` contract.
3. Import this `Constants.sol` contract into all the protocol's contracts and utilize the constants values.

This approach will yield substantial enhancements in code readability and, importantly, serve as a safeguard against potential bugs and errors.

## Missing Functions in the Docs

The documentation, particularly the smart contract documentation and function descriptions, provides valuable insights. However, it's worth noting that there are certain functions currently lacking documentation, here is the list of the most important functions which lack documentation:

1. `NextGenCore.sol`
    1. `airDropTokens`
    2. `mint`
    3. `burn`
    4. `burnToMint`
    5. `_mintProcessing`
    6. `addMinterContract`
    7. `updateAdminContract`
    8. `setDefaultRoyalties`
    9. `getTokenName`
2. `MinterContract.sol`
    1. `mintAndAuction`
    2. `setPrimaryAndSecondarySplits`
    3. `updateCoreContract`
    4. `updateAdminContract`
    5. `emergencyWithdraw`
3. `NextGenAdmin.sol`
    1. `registerBatchFunctionAdmin`
4. Randomizer Contracts
    1. Add documentation for the 3 different randomizer contracts and their functions.

### ✅ Recommendations

I strongly suggest adding documentation for the following important functions in the protocol documentation.

## Poor Contract Storage and Functionality Design

The rationale behind separating the **minter** and **core** contracts remains unclear, as both contain functionalities related to creating and setting collection data, as well as minting and burning tokens. 

This distribution of functions across both contracts complicates understanding the distinct roles of the Minter and Core contracts. Furthermore, due to this division, it's challenging to determine where minting occurs and where the creation and management of collections take place. Clarity and distinction in the contracts' purposes would greatly benefit the codebase.

### Distributed Functions

1. There is a `mint()` function in both the Minter and Core contracts where the `mint()` function within the Core contract can only be accessed by the Minter contract.
2. Setting collection and token data happens both in the `NextGenCore.sol` (`setCollectionData`, `freezeCollection`, `setTokenHash`, `setFinalSupply`, `setDefaultRoyalties`, `updateCollectionInfo`, `changeTokenData`, `updateImagesAndAttributes`, `freezeCollection`, ..) and in the `MinterContract.sol` (`setCollectionCosts`, `setCollectionPhases`, `updateDelegationCollection`, `setPrimaryAndSecondarySplits`, `proposePrimaryAddressesAndPercentages`, `proposeSecondaryAddressesAndPercentages`, `acceptAddressesAndPercentages`)

### Distributed Storage

1. Some of the collection and token data is stored and handled by the `NextGenCore.sol` contract, for instance: `collectionInfo`, `isCollectionCreated`, `wereDataAdded`, `tokenToHash`, `tokenIdsToCollectionIds`, `collectionAdditionalData`, …
    
    And some of it is managed and stored in the `MinterContract.sol` contract, for instance: `setMintingCosts`, `collectionPhases`, ..)
    

### Poor Storage Management

There are numerous unnecessary state variables and mappings in both `MinterContract.sol` and `NextGenCode.sol` contracts. These primarily consist of mappings of `uint256 ⇒ bool`, storing data about the status of collections and tokens.

Here's a list of mappings that can be removed, as their use case can be achieved by checking whether the data exists in another mapping:
**NextGenCore.sol:**

1. The mapping `isCollectionCreated` can be removed, instead a simple check such as `collectionInfo[collectionIndex]._collectionName ≠ “”` can be used.
2. The mapping `wereDataAdded` can be removed, instead a simple check such as `collectionAdditionalData[collectionIndex]._collectionArtistAddress ≠ address(0)` can be used
3. The mapping `artistSigned` can be removed, instead a simple check such as `artistsSignatures[_collectionID] ≠ “”` can be used.
4. Both `onchainMetadata` and `collectionFreeze` mappings can be removed, and the booleans values can be moved of the `collectionAdditionalData` mapping.

**MinterContract.sol:**

1. The mapping `setMintingCosts` can be removed, instead a simple check such as `collectionPhases[collectionIndex].collectionMintCost ≠ 0` can be used.
2. The mapping `mintToAuctionStatus` can be removed, instead a simple check such as `mintToAuctionData[mintIndex] ≠ 0` can be used.

### ✅ Recommendations

1. Revise the `MinterContract.sol` and `NextGenCore.sol` contracts: either merge them into one contract or divide them into smaller contracts, ensuring no redundancy in logic or functionality.
2. Establish a centralized storage for all smart contracts; create a new smart contract, `NextGenStorage.sol`, responsible for managing the protocol storage. Both Minter and Core contracts can utilize its getters and setters for storage management.
3. Streamline the code and conserve gas by eliminating unnecessary mappings.

## No Tests

The existing test coverage for the protocol contracts is insufficient. Some contracts lack tests entirely, while others are deficient in function tests. The images below shows the current test coverage of the codebase using the Hardhat coverage module:

![NextGen-hardhat-test-coverage](https://i.imgur.com/wuWzrwz.png)

Testing smart contracts thoroughly and achieving a 100% test coverage is crucial for making sure they work correctly and are secure. This helps find and fix potential issues early on, reducing the risk of problems later. Full test coverage ensures that every part of the smart contracts is checked. It also makes it easier to maintain and update the protocol.

### ✅ Recommendations

1. Make sure that all functions in all of the contracts are tested. 
2. Create a test scenario from collection creation to full minting process.
3. Test different minting phases, collection types, and minting functionalities
4. Test edge cases and both Good and bad scenarios (Things that should work and things that shouldn’t work)

## Project Structure Could Be Significantly Improved

When working on a project or submitting a project for an audit or for an external developer review, following conventions is crucial. For **Hardhat** projects, it's best to provide the entire project structure with contracts and tests. 

Organizing contracts into subfolders within the `contracts\` directory, distinguishing between core logic, helper contracts, interfaces, and libraries, is  also important. 

In the NextGen repository, the presence of redundant `hardhat` and `smart-contracts` folders containing the same contracts caused confusion. 

Additionally, not organizing smart contracts into these subfolders as recommended led to less organized code management. Following these practices simplifies the audit process and fosters collaboration among developers.

### ✅ Recommendations

Create a single well-organized folder encompassing all project-related files, eliminating the redundancy of having two folders containing the same smart contracts. Additionally, within the `contracts\` directory, further organization can be achieved by categorizing contracts into separate folders according to their respective roles, as previously outlined. 

## Unused `Ownable.sol` Imports

In some of the smart contracts there are imports the `Ownable.sol` contract which is not being utilized, here is a list of all the unused imported contracts in the protocol:

1. The `NextGenRandomizerNXT.sol` contract imports `Ownable.sol` without using it nor inheriting from it.
2. The following contracts import `Ownable.sol` and inherit the `Ownable` contract without using it (there is no use of the `owner` variable or the `onlyOwner` modifier).
    1. `NextGenRandomizerVRF.sol`
    2. `RandomizerRNG.sol`
    3. `MinterContract.sol`
    4. `NextGenCore.sol`

### ✅ Recommendations

Remove all the imports and the inheritance of the `Ownable` contract in all the contracts mentioned above.

## Function Naming Issues

Certain function names could be improved. they don't always effectively reflect the function's purpose and functionality. My suggestion is to meticulously review all functions and align their names more closely with their intended roles. Following this, carry out a comprehensive code refactoring to integrate these new, more fitting function names. Here are a few examples to illustrate:

1. The function `addMinterContract()` in the `NextGenCore.sol` contract refers to setting and updating the minter contract address. A name such as `setMinterContract()` or `updateMinterContract()` is more suitable..
2. The function `addRandomizer()` in the `NextGenCore.sol` contract refers to setting and updating the randomizer for a collection. A name such as `setCollectionRandomizer()` or `updateCollectionRandomizer()` is more suitable..

### ✅ Recommendations

For each function, consider the most fitting name that accurately reflects its purpose and functionality. Proceed to refactor all function names accordingly.

# Access Control and Centralization Risks

![NextGen-access-control](https://i.imgur.com/DZSZXhB.png)

## Access Control Overview

The access control system in the protocol relies on the **NextGenAdmin.sol** contract. This contract comprises three distinct levels of access control:

1. **Global Admin:** This role is endowed with comprehensive access to all functions safeguarded by any modifier within the protocol. The presence of a single Global Admin entails a substantial central point of failure. If this admin's account is compromised, the integrity of the entire access control mechanism and the associated modifiers will be compromised.
2. **Function Admins:** Access may be granted to accounts based on the signature hash or selector of specific functions, adopting a concept employed in balancer smart contracts. Function Admins possess more limited permissions compared to the Global Admin, as their access is confined to pre-approved functions.
3. **Collection Admins:** This role pertains to administrators assigned to specific collections. Each collection is associated with a unique collection ID, allowing for the delegation of access to particular functions based on the collection ID parameter that was passed to the function.

## The `updateAdminContract` Function

Furthermore, within each contract in the protocol, there exists a function for the modification of the admin contract address (`updateAdminContract`). This function effectively enables the "upgradability" of the admin contract. However, this function is safeguarded by the `FunctionAdminRequired` modifier, granting authorization to both the global admin and accounts with specific function access permissions. Nonetheless, this introduces potential vulnerabilities and error-prone scenarios, with two primary risks:

1. **Unintended Updates:** The is the possibility of mistakenly updating the admin contract to a defective or incorrect contract. Such an error would render any contract reliant on the admin contract inoperative, effectively blocking access to all functions protected by modifiers.
2. **Malicious Manipulation:** In a more malicious scenario, an attacker could compromise the global admin account or an account with access to the `updateAdminContract` function selector. Subsequently, they could update the admin contract to a non-functional state, resulting in a breakdown of the protocol. 

## The `updateCoreContract` Function

An additional function susceptible to vulnerabilities and potential errors is the `updateCoreContract` function within the Minter contract. This function serves as another central point of failure in the protocol. A mistake or malicious action could result in an update to a core contract that is non-functional, subsequently causing a complete breakdown in the functionality of the entire protocol. 

## The `addMinterContract` Function

An additional function susceptible to vulnerabilities and potential errors is the `addMinterContract` function within the Core contract. This function serves as another central point of failure in the protocol. A mistake or malicious action could result in an update to a minter contract that is non-functional, subsequently causing a complete breakdown in the functionality of the entire protocol.

**A minor note regarding the function name:** it doesn't accurately convey its intended purpose. Given that there is only one Minter contract, it would be more appropriate to use names like `updateMinterContract` or `setMinterContract` to better align with the function's functionality. 

# Gas Recommendations

Modifying the logic in the functions as suggested below can significantly reduce the gas usage:

## Minter.sol:burnOrSwapExternalToMint()

In the `burnOrSwapExternalToMint()`  the following validation (require statement) can be moved to the top of the function to save gas in case there is issue with the `_tokenId` parameter:

```solidity
require(
    _tokenId >= burnOrSwapIds[externalCol][0] &&
      _tokenId <= burnOrSwapIds[externalCol][1],
    "Token id does not match"
);
```

## AuctionDemo.sol:returnHighestBid()

The [subsequent check](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L75-L76) ensuring that the highest bid status is true can be omitted, as another [redundant check occurs a few lines earlier](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L70) within the for loop. Each bid is already examined for a `status == true`, determining whether it becomes the highestBid and index variables.

## AuctionDemo.sol:claimAuction()

This function is inefficient and may consume a lot of gas. It might **not work properly** when there are many bids due to its design.

The function goes through the bidders array, which is flexible and **can grow** as more people bid on a token. This flexibility is risky because someone could exploit it to launch a Denial of Service (DOS) attack by bidding many times on a token, causing the bids array to expand excessively. This problem was highlighted in the bot report, so I won't go into more details. 

Additionally, the number of for loops can be significantly reduced for better efficiency.

Currently, the function goes through the bidders array four times:

1. It checks the `WinnerOrAdminRequired` modifier using `returnHighestBidder`.
2. Then it calls `returnHighestBid(_tokenid)`.
3. After that, it calls `address highestBidder = returnHighestBidder(_tokenid)`.
4. Finally, the function iterates through the bids again.

This could be made more efficient with just 2 for loops by:

1. Merging `returnHighestBidder` and `returnHighestBid` into a single function that goes through the array once and provides both the highest bidder and bid.
2. Removing the winner check from the `WinnerOrAdminRequired` modifier and using a require statement directly in the `claimAuction()` function.

Another approach could reduce it to just 1 for loop by eliminating the use of `returnHighestBidder` and `returnHighestBid` functions and executing all the logic in a single for loop within the `claimAuction()` function.

## Conclusion

In essence, the Nextgen project holds the potential to be a compelling protocol that captures the interest of the NFT community and addresses intriguing challenges. However, it is crucial to tackle the identified risks and implement measures to safeguard the protocol against potential malicious attacks. A substantial refactor of the protocol implementation and robust testing are important. 

Furthermore, enhancing documentation and code comments is recommended to foster better understanding and collaboration among developers and security researchers.



### Time spent:
50 hours
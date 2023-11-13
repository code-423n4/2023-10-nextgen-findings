## Overview

The NextGen protocol is designed to allow for the efficient creation and management of many unique NFT (ERC721) collections. Collection creators are able to specify and control many unique parameters for their collections including sales types (exponentially or linear price decay, etc.), minting durations (including specifying required time between each mint), whether there is an allowlist (controlled by merkle proofs), and other designations. Additionally, there are unique minting methods which a creator can select from, including standard public/allowlist `mint`, `burnToMint` for minting passes using collections in NextGen, `mintAndAuction` to allow for auctioning each minted NFT, and `burnOrSwapExternalToMint` which is similar to `burnToMint` in implementing logic for minting passes, but using an external ERC721 contract.

#### Contracts

###### There are three core contracts which make up this system:
1. `NextGenCore`: Has logic for setting and retrieving metadata for all collections, including each token's URI. This is also the main contract which stores the ownership information for all collections, as it inherits from the ERC721 OpenZeppelin contract. Each collection created is allocated a maximum of 1e10 unique tokenIds, whose region can be seen by checking a collection's `reservedMinTokensIndex` and `reservedMaxTokensIndex`. As this contract is intended to manage the NFT ownership information, it contains the functions: `airDropTokens`, `mint`, and `burnToMint`, which handle actually minting the NFT tokenIds, while additionally updating relevant metadata according to the mint-type.

2. `MinterContract`: Has logic for (1) setting the fee distribution per collection & paying out fees, (2) setting other metadata for collections including the mint price & mint-type used for a collection (and associated data, such as the `merkleRoot` for the allowlist), and (3) functions for actually minting (`mint`, `burnToMint`, `mintAndAuction`, `burnOrSwapExternalToMint`). This is the contract which minters will interact with in order to actual mint NFTs from a collection, and each minting function will interact with the associated function of the `NextGenCore` contract.

3. `NextGenAdmins`: Has logic for setting and retrieving the each of the unique admin types, which include the universal admins (can generally call all protected functions), function admin (can call specific functions, specified by a function selector), and collection admin (can call collection specific functions).

###### There are also five periphery contracts in this system:
1. `RandomizerNXT`: Integrates with the `XRandoms` contract to return a pseudo-random number.
2. `XRandoms`: Has logic for returning a pseudo-random number & word from an array.
3. `RandomizerVRF`: Has logic for returning a random number using Chainlink VRF.
4. `RandomizerRNG`: Has logic for returning a random number using Arrng.
5. `AuctionDemo`: Has functions for handling the auction of NFTs created from the `MinterContract:mintAndAuction` function. 

#### General flow

###### The general flow (simplified) for creating a collection & users minting from a collection:
1. NextGen admin will call `NextGenCore:createCollection`, which sets metadata for that collection including the collection's name.
2. The collection's admin will call `NextGenCore:setCollectionData`, which sets additional collection metadata, such as the total supply (`collectionTotalSupply`).
3. NextGen admin will call `NextGenCore:addRandomizer`, which will set which randomizer contract to use for that collection.
4. The collection's admin will call `MinterContract:setCollectionCosts`, which sets the mint cost for that collection, along with other data.
5. The collection's admin will call `MinterContract:setCollectionPhases`, which sets the times for the allowlist (and merkle root) and public mints.
6. NextGen admin will call (1) `MinterContract:setPrimaryAndSecondarySplits`, (2) `MinterContract:proposePrimaryAddressesAndPercentages`, (3) `MinterContract:proposeSecondaryAddressesAndPercentages`, (4) `MinterContract:acceptAddressesAndPercentages`, which in aggregate configures the fee distribution for primary and secondary sales of the NFTs.
7. Minter can then call `MinterContract:mint` or other associated minting methods depending on which method the collection uses.

###### Following the end of the minting phase, the general flow (simplified) is:
1. NextGen admin calls `NextGenCore:setFinalSupply`, which will effectively prevent any future minting for that collection.
2. NextGen admin calls `NextGenCore:freezeCollection`, which will prevent any future changes to metadata for the collection.
3. NextGen admin calls `MinterContract:payArtist` to pay out fees to the collection team and artist.

## Architecture Analysis

The overall architecture is for the most part very straight forward. There are set steps which are taken to configure a collection for minting, and handling the steps following the end of minting (finalizing collection data & paying fees). Here I will highlight some issues & areas of improvement.

#### Unnecessary dependencies

The first thing which stands out with the general structure of the NextGen protocol is the amount of unnecessary external dependencies, which effectively moves a lot of trust off-chain, meaning there is a heavier reliance on the NextGen team's admin than is necessary. This is both a burden to the NextGen team, as they will need to manage, for example, secondary fee collection for many different collections, which all happens off-chain. Below are a few examples of this issue & how they could be reasonably solved:

###### (1) Secondary fee collection & distribution happens off-chain 

In the `MinterContract`, the Artist for a collection (or NextGen admin) will call `MinterContract:proposeSecondaryAddressesAndPercentages` to specify which addresses & percentages they want secondary sale fees to go to. However, this is never again used/referenced on-chain, as the protocol team will manage, off-chain, how the fees they collect in secondary sales will be distributed to each collection's team and artists. The issue with this is simple to see, because as the number of collections grows, it will be increasingly difficult for the NextGen team to handle & ensure that the appropriate amount of secondary sales fees are going to each collection. This simply means that there's an increased probability that mistakes can be made, such as for example, teams receiving less royalties than they should be.

There are a few ways in which this can be brought back on-chain and made more trustless. Since the `NextGenCore` contract is already integrated with ERC2981, you could simply allow a collection's admin to change the royalties for their controlled tokenIds to point to their own address, or integrate this with the logic in the `MinterContract`, so that fees can be sent and distributed according to the distributions set in `MinterContract`. Or, at a minimum, have an equivalent function to `payArtist`, where secondary sales fees are placed back on chain, so that they can be distributed according to what has been set in `MinterContract`.

###### (2) Additional unnecessary dependencies, such as in `mintAndAuction`

The logic of the `MinterContract:mintAndAuction` function requires an unnecessary intermediate transfer to some address, which will hold this NFT until the auction is performed in the `AuctionDemo` contract. This firstly requires that this intermediate address correctly approves the `AuctionDemo` contract to transfer that token it has received, along with ensuring that the token is not transferred elsewhere in the meantime. Both of these are unnecessary dependencies, and the `AuctionDemo` contract could be easily updated so that it will take ownership of the NFT during the auction. This allows users to more easily verify that the Auction is safe to use, and that winners will actually receive the NFT.

#### Gas efficiency & safety of design

The second thing which stands out are the numerous places in which the current design has introduced significant gas costs and potential safety issues. For example, the use of safe-transfer calls throughout the contract, such as in `NextGenCore:_mintProcessing`:
```solidity
function _mintProcessing(..) internal {
    ...
    _safeMint(_recipient, _mintIndex);
}
```
or the use of low-level calls, such as in `AuctionDemo:cancelBid`:
```solidity
function cancelBid(uint256 _tokenid, uint256 index) public {
    ...
    (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
    ...
}
```
Without re-entrancy protection, both of these effectively allow for callbacks to all functions which are not role-protected. This leads to multiple potential exploits in this codebase.

Additionally, the current design is very gas-inefficient. For most of the minting types, users can be expected to mint more than a single NFT, meaning minting logic which supports this efficiently would be preferable, such as ERC721A by Solady. The current design will loop over every NFT tokenId the user is attempting to mint, and at the same time perform redundant state variable updates. Generally, there is a lot of improvement which can be made to optimize these set of contracts for gas-usage, and at the same time prevent them from being abused by malicious actors. Especially considering that these contracts are going to be deployed on Ethereum, gas usage should be re-examined.

## Codebase Analysis

For the most part the codebase was understandable and I could follow the general flow for collection creation & minting which is outlined in the docs. Here I will highlight some issues & areas of improvement.

#### Code quality & readability

Throughout the codebase there is a general lack of comments which explain what functions are intended to do (along with what variables are intended to represent). There is also a general lack of formatting & a lack of natspec comment structure, which make it harder to understand the codebase. Much of this information is included in the docs, but it would greatly improve readability to also include this information as comments in the code itself.

The use of full addresses such as in [MinterContract](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L205), rather then e.g. using `address(0)` make the codebase less readable. Additionally, there is inconsistent usage of getter functions vs simply referencing mappings directly throughout the codebase which make it less readable. For example, there is inconsistent usage of the `NextGenCore:viewColIDforTokenID` getter function vs simply directly getting `tokenIdsToCollectionIds[_tokenid]`. One or the other should be used exclusively. Finally, certain functions are left public when they should be internal or private, such as `RandomizerRNG:requestRandomWords`, as these functions should only ever be called internally. This adds unnecessarily to the attack surface of their respective contracts.

#### Code safety

There is a lack of input sanitation in certain places. For example in `NextGenCore:setCollectionData`, they don't check that `_collectionTotalSupply` is not 0, which can result in overriding certain collection parameters which should not be possible in a follow up call.

There is also generally a large issue with allowing arbitrary callbacks in functions throughout this codebase, and at the same time a general lack in reentrancy protection for all functions. All the safe-transfer calls including in [NextGenCore](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L231), and the low-level calls for example in [AuctionDemo](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L128) allow for callbacks, which malicious users can abuse to break the intended logic. Re-entrancy guards should be added to all unprotected functions in this codebase, such as `MinterContract:mint`, `MinterContract:burnToMint`, `AuctionDemo:cancelBid`, `AuctionDemo:cancelAllBids`, etc.

Some additional issues include a general lack of managing array length which is looped over in the `AuctionDemo`contract and can result in out-of-gas issues if there are too many bids for a given collection. There is also precision loss which occurs in certain places in the codebase due to division before multiplication. One such example is in the [MinterContract](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536).

#### Code efficiency & gas costs

There are significant improvements which can be made to decrease the gas costs for the minting logic in this contract. The minting of >1 tokens per user in this protocol requires looping over every mint of a tokenId and minting the tokens individually. Considering it is the general design of this system that users will be minting more than a single token, an implementation similar to Solady's ERC721A contract might be more prudent, and prevent skyrocketing gas costs. There are also multiple places in this codebase where excess gas is required due to poor logic. One such example in in the [NextGenCore](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L191) contract in the `mint` function:
```solidity
function mint(...) external {
    require(msg.sender == minterContract, "Caller is not the Minter Contract");
    collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
    if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
        _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
        ...
    }
}
```
As already mentioned, with most of the minting functionality, it is expected that users will often be minting more than a single NFT. In the above logic, the `collectionCirculationSupply` of a collection is updated every time that an NFT is minted for a user, rather than simply updating this amount a single time based on the aggregate amount a user is minting in that call. This effectively results in an excessive additional n-1 sstore operations for every n amount minted by a user, which is a very gas-intensive operation.

Additionally, there are many cases in which unnecessary storage of data which can be easily inferred is done. For example, there is a separate mapping of [tokenId->collectionId](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L230) that is stored, which is unnecessary considering that it is trivial to infer which collection a given tokenId is a part of. Similarly, there is a general lack of caching and re-using of values throughout the codebase which results in a lot of unnecessary gas costs. For example, in `MinterContract:burnToMint`, the index calculated for the next token to mint is done [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L264) and [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267). As this is done twice, this is an unnecessary waste of gas, and in general the codebase should be refactored to remove these inefficiencies.

#### Code redundancy 

There are many places in this codebase where there is either logic which has no practical impact or is completely unnecessary. For example, the `NextGenCore:addRandomizer` function has a check to determine whether some contract is a legitimate randomizer contract by checking its return value from the function call to `isRandomizerContract`. However, when checking how this is implemented, each randomizer contract simply includes the following function:
```solidity
function isRandomizerContract() external view returns (bool) {
    return true;
}
```
Since this check does not actually do anything, this check is completely redundant, as for example, and fraudulent variants of randomizer contracts can just add the same function definition to circumvent this check. This particular redundant check occurs in all functions for setting contract addresses, including in `NextGenCore:updateAdminContract`.

There are also other occurrences of redundant checks, where effectively the same checks are done multiple times. Although generally double checking is at a minimum not a negative, this ultimately adds to the total gas costs. One such example of this includes all the minting functions on the `NextGenCore` contract, such as the `mint` function, which has the following check on [line 192](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L192): 
```solidity
if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
	...
}
```
This check effectively ensures that the minted tokenId for a user does not exceed the total supply for that collection. However, this check is already done in the `mint` function of the `MinterContract` on [line 232](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L232):
```solidity
require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```

Effort should be taken to remove these redundant checks and logic, as they simply add unnecessary complexity to the codebase, while at the same time increase the gas costs of calling these functions.

#### Validation of input & function calls

Across the codebase there is a general lack of validation of the values set for core variables. As a simple example, there is a lack of checking that `allowlistEndTime` < `publicStartTime` in the case where these are intended to be distinct periods - also there is no check that `_auctionEndTime` in `MinterCOntract:mintAndAuction` is actually in the future, and not for example 0, which would break the logic for the auction. 

Likewise, there are certain sanity checks which are only performed in certain places, rather than simply being done in all functions which have similar logic. One example of this is that `MinterContract:burnToMint` does not check that the price for the collection has been set, while this check is correctly done in the `MinterContract:mint` function call. There is no reason for these inconsistencies, and they should be remedied, as they add confusion and increase the probability that errors will be made.

## Centralization Analysis

There is a huge centralization component to the NextGen protocol as a significant amount of the functionality is gated behind role-based access, in which the NextGen protocol admin have control over effectively everything. Even functions such as `NextGenCore:updateImagesAndAttributes` and `NextGenCore:setCollectionData` (which are entirely collection dependent) can be called and altered by the NextGen admin. This effectively means that there are huge trust assumptions being made about the address representing the NextGen admin. Note that this includes both trusting the NextGen team inherently & trusting that there are no issues with the safety of their SecOps.

This is due to the definition of roles, such as the following for the collection admin:
```solidity
    modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
      _;
    
```
As can be seen, the collection admin role allows for both the function admin and NextGen admin role as well. There are no functions which are solely designated to the collection admin.

Frankly, I believe that it is not always necessary for the NextGen admin to have the ability to call functions which are clearly only relevant for specific collections. Rather, these should only be called by the collection admin and no one else. One such example is `NextGenCore:setCollectionData`, which has functionality for setting data including the total supply for the collection and the collection artist's address. Reasonably, this is something which should under no circumstances be altered or ever called by the NextGen admin, and allowing them to do so adds an unnecessary attack surface to these contracts.

Additionally, many of the roles related to allowing certain addresses to call specific functions are often configured for functions which should only be called by the collection's admin. More generally, functions which alter the data or state related to a specific collection should only be callable by that collection's admin. The function admin as defined in the `NextGenAdmins` contract in general have too many rights. One such example is `MinterContract:setPrimaryAndSecondarySplits`, which sets the team's and artist's split of the royalties/fees from a collection. This is only callable by the admin who has access to calling that specific function (the collection admin can't call this). This means that admin has the power to arbitrarily alter the data across all collections, which is excessive, and requires significant trust on the part of the collections that these admins will not abuse their power. Another example is the `MinterContract:payArtist` function, where the function admin has power over what percentage of the fees will go to `_team1` and `_team2`. This selection should reasonably only be made by the collection's admin.

In addition to arguably unnecessary excessive rights given to certain admin, there is generally just an overarching theme of excessive trust which is necessarily given to the NextGen team. Consider the `MinterContract:emergencyWithdraw` function:
```solidity
function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
    uint balance = address(this).balance;
    address admin = adminsContract.owner();
    (bool success, ) = payable(admin).call{value: balance}("");
    emit Withdraw(msg.sender, success, balance);
}
```
The MinterContract holds all of the primary collection fees, and there is a single function which is able to drain the entire contract of all funds. This means that any compromised keys for the admin multisig could result in draining the entire contract.

In general this is not to say that there are any implicit issues with trusting the NextGen team. However, I believe that it is best to minimize trust assumptions wherever possible, and there are certainly many areas across this protocol which can be improved in that respect. I have already highlighted many of them, but generally, admin rights should be restricted as much as possible, which is not done in the current implementation.

## Systemic Risks Analysis

Systemic risks generally relate to integrations of a protocol with other external protocols/data sources, which is something this protocol minimizes. The only external sources this contract interacts with are random number generators. However, some of these integrations do have some potential issues. Although not an interaction with an external source, the randomness logic of the `RandomizerNXT` contract relies on the `XRandoms` contract, which in turn uses `block.prevrandao`. However, this value is known ahead of time when a user creates a minting tx, meaning they could potentially game this randomness logic to get an optimal mint.

However, there are also potential issues with the integrations with external randomness sources. For example, the integration with Arrng in `RandomizerRNG` relies on the admin constantly updating the `ethRequired` variable, which specifies the amount of ETH required to pay for the randomness Arrng service. The problem is that during times of high congestion, it's possible that the actual callback cost will exceed the stored `ethRequired` amount stored by the admin, which will then result in the payment for the service being less than what's required. This in turn will results in the call (not reverting) never returning a random value. There is also no logic for re-triggering a call to get a random number for a minted tokenId if the original call failed, meaning this randomness will then never be set for that token.

## Approach Taken

I began by examining the core functionality implemented in the `NextGenCore`, `MinterContract`, and `NextGenAdmins` contracts. I started by reviewing (1) the flow for setting up contracts for minting, (2) then the minting logic itself, and finally (3) the logic post-minting, for finalizing the metadata of a collection & other related logic. Following this, I reviewed the logic for generating randomness in the `RandomizerNXT`, `XRandoms`, `RandomizerVRF`, and `RandomizerRNG` contracts. Finally, I reviewed in detail the auctioning logic of the `AuctionDemo` contract. In hindsight I believe this was the best approach to take, as it was important to understand the core contracts in order to understand the importance/usage of the periphery contracts.

## Conclusion

The NextGen protocol has designed an interesting approach for enabling multiple unique collections to mint their NFTs from the same contract, including allowing for unique minting methods such as burn-to-mint passes. There are however certain issues with the current implementation, including excessive external dependencies which are not necessary, increasing the trust assumptions required both by the collections themselves, along with minters. Additionally, there is a lack of safety measures to prevent reentrancy, along with currently a non-optimal design for gas-efficient minting, which will result in excess gas costs. Furthermore, there are significant trust assumptions required for this protocol, as much of the functionality is based on role access, in which the NextGen admin effectively always has the ability to override. With select changes to areas I have highlighted, I believe the NextGen protocol could be much stronger.

### Time spent:
20 hours
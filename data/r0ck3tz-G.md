# Gas Optimization

## Summary 

| Id     | Title                                                                           |
|--------|---------------------------------------------------------------------------------|
| [G-01] | Pack structs                                            |
| [G-02] | Expensive functions `returnHighestBid` and `returnHighestBidder`                    |
| [G-03] | Obsolete code                                         |
| [G-04] | Storing the same value multiple times                                           |
| [G-05] | Use constant value for random words                                           |
| [G-06] | Inheriting contracts that are not used                                            |
| [G-07] | Cache storage values                                        |
| [G-08] | Unused storage variables                                           |
| [G-09] | Use `uint256` type instead of boolean                                      |
| [G-10] | Obsolete checks in `NextGenCore` functions                                         |
| [G-11] | Optimize the delegation allowance                                     |
| [G-12] | Unnecessary calculation                                     |

## [G-01] Pack structs

- In `AuctionDemo.sol` struct [`auctionInfoStru`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L43-L47) stores auction data in 3 storage slots. This can be easily packed into a single slot, by changing `uint256 bid` to `uint88` which can handle  over `309485009 ether`:
```solidity
struct auctionInfoStru {
    address bidder;
   uint88 bid;
   bool status;
}
```
- In `MinterContract.sol` struct [`collectionPhasesDataStructure`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L44-L56) uses 10 storage slots but can be reduced to 4 storage slots:
```solidity
struct collectionPhasesDataStructure {
    uint32 allowlistStartTime;
    uint32 allowlistEndTime;
    uint32 publicStartTime;
    uint32 publicEndTime;
    uint32 timePeriod;
    uint32 rate;
    bytes32 merkleRoot;
    uint128 collectionMintCost;
    uint128 collectionEndMintCost;
    uint8 salesOption;
    address delAddress;
}
```
- In `MinterContract.sol` struct [`royaltiesPrimarySplits`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L63-L66) holds percentages. These values are small and both can be held in a single storage slot instead of two:
```solidity
struct royaltiesPrimarySplits {
    uint128 artistPercentage;
    uint128 teamPercentage;
}
```
- In `MinterContract.sol` struct [`collectionPrimaryAddresses`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L73-L81) uses 7 storage slots but it can be reduced to 3 storage slots:
```solidity
struct collectionPrimaryAddresses {
    address primaryAdd1;
    uint88 add1Percentage;
    address primaryAdd2;
    uint88 add2Percentage;
    address primaryAdd3;
    uint88 add3Percentage;
    bool status;
}
```
- In `MinterContract.sol` struct [`royaltiesSecondarySplits`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L88-L91) uses two storage slots but it can be reduced to one storage slot:
```solidity
struct royaltiesSecondarySplits {
    uint128 artistPercentage;
    uint128 teamPercentage;
}
```
- In `MinterContract.sol` struct [`collectionSecondaryAddresses`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L98-L106) uses 7 storage slots but it can be reduced to 3 storage slots:
```solidity
struct collectionSecondaryAddresses {
    address secondaryAdd1;
    uint88 add1Percentage;
    address secondaryAdd2;
    uint88 add2Percentage;
    address secondaryAdd3;
    uint256 add3Percentage;
    bool status;
}
```
- In `NextGenCore.sol` struct [`collectionAdditonalDataStructure`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54) uses 9 storage slots but it can reduced to 4 slots:
```solidity
struct collectionAdditonalDataStructure {
    address collectionArtistAddress;
    uint32 maxCollectionPurchases;
    uint32 collectionCirculationSupply;
    uint32 collectionTotalSupply;
    uint256 reservedMinTokensIndex;
    uint256 reservedMaxTokensIndex;
    uint64 setFinalSupplyTimeAfterMint;
    address randomizerContract;
}
```

## [G-02] Expensive functions `returnHighestBid` and `returnHighestBidder`

The `AuctionDemo` contract implements functions `returnHighestBid` and `returnHighestBidder` that dynamically search for the highest bid and bidder. This is very inefficient. Consider adding storage values `highestBid` and `highestBidder` that are updated on new bid and cancel bid functionality

## [G-03] Obsolete code

- In `AuctionDemo.sol` the [additional check](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L75-L79) in `returnHighestBid` is not needed and can be removed. This is because there is already check for the `auctionInfoData`'s `status` and `highBid` has initial value of `0`.
- The `proposeSecondaryAddressesAndPercentages` sets the value of status `collectionArtistSecondaryAddresses[_collectionID].status` to `false` which is obsolete since that value is `false` already.
- The `proposePrimaryAddressesAndPercentages` sets the value of status `collectionArtistSecondaryAddresses[_collectionID].status` to `false` which is obsolete since that value is `false` already.

## [G-04] Storing the same value multiple times

- The `NextGenRandomizerNXT` and `NextGenRandomizerRNG` store the address of `NextGenCore` in `gencore` and `gencoreContract` storage variables. These values are exactly the same and stored in the form of the address in the storage. It is recommended to store the value of the `gencore` address once and then use an interface to interact with it.
- The `NextGenCore` contract stores the same value in the `collectionAdditionalDataStructure` structure of `randomizerContract` and `randomizer`. It is recommended to store the value of the `randomizer` address once and then use an interface to interact with it.

## [G-05] Use constant value for random words

The `wordList` is being constructed in memory within `getWord` of `randomPool` contract every single time the function is triggered. Consider moving `wordList` outside of the function as constant values.

## [G-06] Inheriting contracts that are not used

- Contract `NextGenRandomizerRNG` inherits from `Ownable` contract that functionality is not used. It is recommended to remove inheritance from `Ownable` contract.
- Contract `NextGenRandomizerVRF` inherits from `Ownable` contract that functionality is not used. It is recommended to remove inheritance from `Ownable` contract.

## [G-07] Cache storage values

Throughout the codebase, there are multiple instances where storage variables are being read repeatedly, leading to inflated gas costs. It is recommended to cache storage variables that are read more than once.

## [G-08] Unused storage variables

There are unused storage variables that should be removed:
- Storage variable `tokenToRequest` mapping in `NextGenRandomizerRNG`.
- Storage variable `tokenToRequest` mapping in `NextGenRandomizerVRF`.

## [G-09] Use `uint256` type instead of `boolean`

There are multiple contracts that are using `boolean` storage values. In case the `boolean` is the only value in the storage slot, it's better to use `uint256` to avoid masking to extract the `boolean`:

1. In the `AuctionDemo` contract, the `auctionClaim` mapping.
2. The `NextGenAdmins` contract is using `boolean` values to represent permissions in mappings. This consumes additional gas since for every comparison it requires masking the 32 bytes value to extract a single-byte boolean value. It is recommended to use uint256 values and check if the value is not 0.
3. The `NextGenCore` contract is using booleans for mappings of `onchainMetadata`, `collectionFreeze`, and `artistSigned`.
4. The `MinterContract` is using booleans for mappings of `burnToMintCollections`, `burnExternalToMintCollections`, and `mintToAuctionStatus`.

## [G-10] Obsolete checks in `NextGenCore` functions

The `airDropTokens`, `mint`, `burnToMint` functions of `NextGenCore` contract implement following check:
```solidity
if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply)
```

This check is not needed since the correctness of minting tokens is already being verified in the `MinterContract` in the `airDropTokens`, `mint`, and `burnToMint` functions through a check similar to this
```solidity
collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```
It is recommended to remove obsolete checks in `NextGenCore` contract.

## [G-11] Optimize the delegation allowance

The `if` statement checking if `isAllowedToMint` is still `false` in the `mint` and `burnOrSwapExternalToMint` functions is not necessary. It is recommended to use logical or statements to efficiently determine the value of isAllowedToMint.
```solidity
isAllowedToMint = dmc.call1() || dmc.call2() || dmc.call3() | dmc.call4()
```

## [G-12] Unnecessary calculation

The `burnToMint`, `mintAndAuction`, and `burnOrSwapExternalToMint` functions of `MinterContract` are calculating the same value twice. Initially, they calculate `collectionTokenMintIndex`, run some checks, and then recalculate that value and assign it to `mintIndex`. It is recommended to use `collectionTokenMintIndex` for the `mintIndex`.

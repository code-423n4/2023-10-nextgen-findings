### Input validation is needed in `returnIndex`
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L45
We should probably use `id%100` in `returnIndex`, otherwise it'll revert with "out of bounds" error if `id` is > 100.

### If the artist signs collection with total supply 0, a new artist will be unable to sign it
In `setCollectionData` admin can specify different collection parameters, let's look at this rare case:
- admin specifies `collectionArtistAddress` but lefts `collectionTotalSupply` empty,
- artist signs this collection 
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L261

- admin decides to set a new collectionAddress, it is implied in check 
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L149
```
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
```

that if `collectionTotalSupply == 0` it is possible to change the artist,
- new artist won't be able to sign because `artistSigned` was already set by the previous artist
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L259

### `isXContract` checks provide false sense of security
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L89
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L105
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L62
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L83

when the admin decides to set a new randomizer or admin contract, we call one of the checks above and proceed if it returns `true`, 
`require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");`
this way we make sure that the contract is correct, however it's very easy for an attacker to spoof this check if he wants to use a malicious contract. So in the end we rely only on the admin to provide a "good" contract.

### Sale modes validation needed
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157
In `setCollectionCost` admin can specify multiple sale parameters, minting cost, rate, time period etc. Depending on a sale mode some parameters may be used and some not, for example if we want a fixed cost sale we can specify `rate` and `timePeriod` as zero because they aren't used in the price calculations, but we use them in other modes, for instance `rate` is used in sale mode 3
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536
and if it was set to zero we'll be unable to get the price of the token. Consider validating parameters depending on what `salesOption` was specified for this collection. 

### `collectionPhases` parameters validation is needed for collection in `initializeBurn`
In `initializeBurn` the admin can specify a collection which tokens users can burn in order to mint tokens from another collection. Only the basic data validation is performed
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L309
however, later in `burnToMint` we use parameters from `collectionPhases` such as public sale timings
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L260
which may be uninitialized at this time. Consider checking that `_mintCollectionID` has it's phase parameters initialized, when calling `initializeBurn`
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L170-L177

### `salesOption` parameter validation is needed for collection in `initializeBurn`
Consider checking `salesOption` of the collection before using it in `initializeBurn` or `initializeExternalBurnOrSwap`, collection with sale mode 3 may be specified as `mintCollectionID`, which will break 1 token per period minting rule as users will be able to mint this tokens without any amount and period restrictions.
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326-L365
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L258-L272

### Check if `_burnOrSwapAddress` != address(0)
In `initializeExternalBurnOrSwap` the admin can specify `burnOrSwapAddress` that users can transfer external NFTs to in exchange for `mintCollectionID` tokens, 
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L340
`IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);`

if `burnOrSwapAddress` is a zero address, transaction above will revert.

### User will pay full minting price at the last second of the public sale
Consider a sale with an exponential price decrease, user calls `mint` function in the last second of the public sale, `block.timestamp = collectionPhases[col].publicEndTime`
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L221
meanwhile in `getPrice`, there is stricter condition `block.timestamp < collectionPhases[col].publicEndTime`
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L540
, and if `block.timestamp == collectionPhases[_collectionId].publicEndTime`, `collectionMintCost` will be returned instead of `collectionEndMintCost`

### There is no way to change minter contract in `AuctionDemo.sol` other than redeploy
In `NextGenCore` admin can set a new minter contract
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L315
`AuctionDemo` on the other hand lacks this functionality and uses minter address that was provided during deployment, as a result in order to use the new minter we'll have to redeploy the contract. Consider adding minter contract setter to `AuctionDemo`.

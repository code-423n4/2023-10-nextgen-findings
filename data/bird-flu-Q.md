# [L1] Reentrancy is possible in MintContract:mint() and MintContract:burnToMint()

Note: This is the same issue as [L5 from the bot report](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a#l-05-functions-calling-contracts-with-transfer-hooks-are-missing-reentrancy-guards), but the report did not catch all instances of this issue.

## Summary
An argument, `address _mintTo`, passed to `MinterContract:mint()` opens up the possibility of reentrancy. `_mintTo` is eventually passed to `ERC721:_safeMint()`, which calls `IERC721Receiver(to).onERC721Received` on the target contract. This `to` contract can then, reenter `MintContract:mint()` before the first call finishes. This is similar to `MintContract:burnOrSwapExternalToMint()` too.

This is also true for `MintContract:burnToMint()`, except that the recipient address is not passed in by the user. The owner of a token is used, but that could also be a malicious contract.

While not posing an obvious risk due to most effects completing before the possible reentrancy, this still leaves an open door to this style of attack.

Instances not found by the automated report:

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L231
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326


## Recommendation
Similar to the recommended from the automated finding, add a reentrancy guard modifier to any functions that have this interaction and are user-facing.

# [L2] Using block based inputs to generate randomness is open to manipulation

**Effected Contract:** RandomizerNXT.sol, XRandoms.sol

## Summary
Using block.number and block.timestamp as a source of randomness is commonly advised against, as the outcome can be manipulated by callers and miners.

Instance:
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L35-L43

## Recommendation
There are other VRF contracts in-scope already. I recommend using those for production, otherwise randomness can be gamed to get optimal token hashes.

# [QA1] All funds in MinterContract are vulnerable to a single leaked key

**Effected Contract:** MinterContract.sol

## Summary
The function `emergencyWithdraw` has a high risk profile for the protocol. If the particular private key with the function admin role `emergencyWithdraw` is exposed, anybody who has the key could drain all eth from the contract. The risk profile for this reaches **all** collections in the protocol.

To reduce the risk profile of the exposure of a single key, it's recommended to use OpenZeppellin's `Pausable` library. With this extension, the Minter can be paused by one signer, with recovery functionality only runnable when the contract is paused, accessible only by separate signers. This 2-step pause-and-recover functionality is much harder to socially engineer. An attacker would need to expose 2 or more private keys to be able to access emergency functionality.

[OpenZeppellin Pausable Library](https://docs.openzeppelin.com/contracts/2.x/api/lifecycle)

Instances:
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L461

## Recommendations
1. Add a Pausable function to the MinterContract with a modifier that only allows a user with `Pausable` role to call it. Add the modifier `whenPaused()` to the `emergencyWithdraw` function.

# [QA2] Rate and TimePeriod set to 0 results in a divide by 0 error in getPrice, temporarily DOS'ing mint functions

**Effected Contract:** MinterContract.sol

## Summary
Many functions in MinterContract use `_rate` and `_timePeriod` as a denominator in math calculations. There is no validation for either of those values in `setCollectionCosts`, so there is potential for those values to be zero. If either of them were 0, using these values as a denominator would result in a 'divide by 0' error and revert, preventing any minter from successfully executing functionality until these values were later set.

Locations that can result in a divide by 0 error:
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L249
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L292
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L536
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L546
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L551
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L553

## Recommendations
1. Add validation to setCollectionCosts.

[MinterContract L157](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L157)
```diff
function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+       require(_timePeriod > 0, "_timePeriod must be greater than 0");        
+       require(_rate > 0, "_rate must be greater than 0");        
```

# [QA3] setFinalSupplyTimeAfterMint can be reset after totalSupply is set

**Effected Contract:** NextGenCore.sol

## Summary
[The comment](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L145) above [`setCollectionData`](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L147) states 'only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set'. However, setFinalSupplyTimeAfterMint can also be set after the final supply is set. This value is later used in [`setFinalSupply`](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L307) to determine if the totalSupply and reservedIndex can be reduced at this time after a mint has completed. Although both functions are accessible only through trusted roles, this functionality could realistically be accidentally hit.

There are potential opportunities for a rogue CollectionAdmin signer to change this information to prevent `setFinalSupply` from executing, but since it's a trusted role, I won't dive into it. I will propose a situation that might arise from the comment above the function not actually matching behavior.

Suppose a CollectionAdmin set the collection data with `setFinalSupplyTimeAfterMint == 10 days`, but then runs the `setCollectionData` again, but passes `0` as a param for `setFinalSupplyTimeAfterMint`, believing the comment above `setCollectionData`: that the value can't be changed again after totalSupply is set. The resulting behavior is that a function admin could then call setFinalSupply sooner than the CollectionAdmin intended.

Instances:
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L161
- https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L164

# Proof Of Concept
```solidity
it("#setCollectionData1", async function () {
    await contracts.hhCore.connect(signers.addr1).setCollectionData(
    1, // _collectionID
    signers.addr1.address, // _collectionArtistAddress
    2, // _maxCollectionPurchases
    10000, // _collectionTotalSupply
    0, // _setFinalSupplyTimeAfterMint
    )
})

it("#setCollectionData1ForSetFinalSupplyTime", async function () {
    await contracts.hhCore.connect(signers.addr1).setCollectionData(
    1, // _collectionID
    signers.addr1.address, // _collectionArtistAddress
    2, // _maxCollectionPurchases
    10000, // _collectionTotalSupply
    100, // _setFinalSupplyTimeAfterMint
    )
})

it("#retrieveCollectionAdditionalDataForCol1", async function () {
    const collectionData = await contracts.hhCore.connect(signers.addr1).retrieveCollectionAdditionalData(
    1
    );

    expect(collectionData[4]).to.equal(100n);
})
```

## Recommendation Options
1. Create a separate function for updating collection data, where the user can only pass the modifiable parameters. This makes it much clearer what you can and can't update, rather than a comment which might fall out of date. If setFinalSupplyTimeAfterMint **should** be updateable, then add it to the update function. If not, then do not add it to the function.
```solidity
function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
    require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
    require(wereDataAdded[_collectionID] == false, "err/dataAlreadySet");
    collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
    collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
    collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
    collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
    collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
    collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
    wereDataAdded[_collectionID] = true;
}

function updateModifiableCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
    if (artistSigned[_collectionID] == false) {
        collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
        collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    } else {
        collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    }
}
```
2. Update the comment to include _setFinalSupplyTimeAfterMint or disallow setFinalSupplyTimeAfterMint to be reset after totalSupply is set.
# QA 

## Use solidity linting for better code readability. 
## Use constant values instead of magic values
1. `_setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);`  
[NextGenCore.sol#L111](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L111)
2. Use constant to specify maxTotalSupply.
[NextGenCore.sol#L148](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L148)
[NextGenCore.sol#L155](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L155)
[NextGenCore.sol#L156](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L156)

# L1 - `setFinalSupplyTimeAfterMint` can also be changed after totalsuplly is changed. 

## Impact 
`setFinalSupplyTimeAfterMint` can also be changed after totalsuplly is changed. That
goes against the comment specification for `NextGenCore.sol::setCollectionData()`. 

## PoC 
`
// function to add/modify the additional data of a collection
// once a collection is created and total supply is set it cannot be changed
// only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set

function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
  require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
  if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
    collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
    collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
    collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
    collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
    collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
    collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
    wereDataAdded[_collectionID] = true;
  } else if (artistSigned[_collectionID] == false) {
    collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
    collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
  } else {
    collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
    collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
  }
}
`

## Mitigation 

Update comment specification or update code to remove `setFinalSupplyTimeAfterMint`
after totalsupply is changed.  
 

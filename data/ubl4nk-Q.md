### `_collectionTotalSupply` should be always passed
```solidity
function setCollectionData(
        uint256 _collectionID, 
        address _collectionArtistAddress, 
        uint256 _maxCollectionPurchases, 
        uint256 _collectionTotalSupply, 
        uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) 
    {
        require((isCollectionCreated[_collectionID] == true) && 
                (collectionFreeze[_collectionID] == false) && 
                (_collectionTotalSupply <= 10000000000), "err/freezed");
```
The condition `(_collectionTotalSupply <= 10000000000)` should be removed from this line and move it some lines after, change the code like this:
```solidity
    function setCollectionData(
        uint256 _collectionID, 
        address _collectionArtistAddress, 
        uint256 _maxCollectionPurchases, 
        uint256 _collectionTotalSupply, 
        uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) 
    {
        require((isCollectionCreated[_collectionID] == true) && 
                (collectionFreeze[_collectionID] == false) , "err/freezed");
        
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) { // @audit totalSupply can not change after setting totalSupply
            require((_collectionTotalSupply <= 10000000000), "Some error");

```
There is no need to check `_collectionTotalSupply` while the `collectionTotalSupply` is already set.
It leads for collectionAdmin to get a revert as-long-as he doesn't set `_collectionTotalSupply`.
`collectionTotalSupply` can be set only once, so what's the need to check `_collectionTotalSupply` every time the `setCollectionData` is called ?! That condition is useless to check every time the admin is passed that agument or not.

Related links:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148
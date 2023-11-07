### 1. `_collectionTotalSupply` should be always passed
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
```diff
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -145,8 +145,9 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set

     function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
-        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
+        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "err/freezed");
         if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
+                       require(_collectionTotalSupply <= 10000000000, "_collectionTotalSupply should be less than (or equal to) 10000000000");
             collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
             collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
             collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;

```
There is no need to check `_collectionTotalSupply` while the `collectionTotalSupply` is already set.
It leads for collectionAdmin to get a revert as-long-as he doesn't set `_collectionTotalSupply`.
`collectionTotalSupply` can be set only once, so what's the need to check `_collectionTotalSupply` every time the `setCollectionData` is called ?! That condition is useless to check every time the admin is passed that agument or not.

Related links:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148

### 2. Add a function that returns the Index of latest token minted in collection
1. Add a function in NextGenCore contract:
```solidity
// function to return latest index id of a collection
function retrieveLatestIndexOfMintedToken(uint256 collectionID) external view returns(uint256) {
   return viewTokensIndexMin(collectionID) + viewCirSupply(collectionID);
}
```
2. Remove this code from MinterContract (or where ever it is seen) (e.g. in MinterContract this code is repeated 10 times):
```solidity
gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID)
```

3. Use this call instead:
```solidity
gencore.retrieveLatestIndexOfMintedToken(collectionID);
```
Related links: 
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L185
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L188
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L231
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L235
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L264
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L359
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L362
### 3. Consider adding functions for checking mint phases
1. Add these 2 functions in MinterContract:
```solidity 
function isPhase1(uint256 col) external view returns(bool) {
    return block.timestamp >= collectionPhases[col].allowlistStartTime &&
            block.timestamp <= collectionPhases[col].allowlistEndTime;
}

function isPhase2(uint256 col) external view returns(bool) {
    return block.timestamp >= collectionPhases[col].publicStartTime && 
                    block.timestamp <= collectionPhases[col].publicEndTime;
}
```
2. Replace this functions where ever you want to check mint phase, for example:
```solidity
        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) 
        {
```
can be replaced with:
```solidity
if (isPhase1(col)) 
{
```
Related links:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L345
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L351
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L260
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L221
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L202

### 4. `mintIndex` is calculated twice in `burnToMint` and `burnOrSwapExternalToMint`
For example in `burnToMint` it is defined once [here:](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L264C9-L264C125)
```solidity
collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
```
and once [here:](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267)
```solidity
uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
```
**They are exactly the same thing and the second one should be removed.**

Related links:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267C9-L267C118

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L359
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L362
- `_collectionID` is 0 at start so that sets `reservedMinTokensIndex = 0`

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L155

- The two assignments shown below are executed regardless of condition in if-else, so it would be recommended to put them aside out side of if-else.

```solidity
collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenCore.sol#L162-L165

- Missing existance check of `_tokenId` using `isCollectionCreated` 
       
`require(isCollectionCreated[tokenIdsToCollectionIds[_tokenId]] == true, "Not Allowed");`

```solidity
    function changeTokenData(uint256 _tokenId, string memory newData) public FunctionAdminRequired(this.changeTokenData.selector) {
        require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");
        _requireMinted(_tokenId);
        tokenData[_tokenId] = newData;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L273-L277

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281-L288

- No emit event on freezing and should revert: `if collectionFreeze[_collectionID] == true;`

```solidity
    function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col");
        collectionFreeze[_collectionID] = true;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292-L295

- Since return is boolean with address param, the function names should be changed to start with 'is' like `isAdmin()`, `isFunctionAdmin()`, `isCollectionAdmin()`in NextGenAdmins.sol

- Contract check functions like these accross the protocol can be restricted to pure since they don't use any storage variable.

```solidity
function isAdminContract() external view returns (bool) {
    return true;
}
```

- Hardcoded address is set in `_setDefaultRoyalty` of constructor when it is recommended to pass via params.

```solidity
   _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L111-L112
The function freezeCollection needs an event emission to notify external listeners that the collection has been frozen. This could be important as it will provide/maintain transparency to users. 

![link] https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292

```solidity 
    function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col");
        collectionFreeze[_collectionID] = true;
    }

```
An event definition could be added in the following way. 

```solidity 
event CollectionFrozen(uint256 indexed collectionId);

function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
    require(isCollectionCreated[_collectionID] == true, "No Col");
    collectionFreeze[_collectionID] = true;
    emit CollectionFrozen(_collectionID);
}
```

As this will emit an event with the CollectionFrozen signature and _collectionId parameter, which can used/tracked by external listeners when collection is frozen 


##### 

A More Clear Descriptive Message Can Be Written 

The error message in multiple lines is not very descriptive. It would be better to have a more descriptive error message, 

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L239-L240
```solidity
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```
Could be changed to 
```solidity
require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "This collection has not been created yet or is currently frozen");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L258-L259

```solidity 
 require(msg.sender == collectionAdditionalData[_collectionID].collectionArtistAddress, "Only artist");
        require(artistSigned[_collectionID] == false, "Already Signed");
```
These could be changed to 

```solidity
require(msg.sender == collectionAdditionalData[_collectionID].collectionArtistAddress, "Only the artist of this collection can perform this action");
require(artistSigned[_collectionID] == false, "This collection has already been signed by the artist");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L267-L268

```solidity 
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```

Could be changed to 
```solidity 
require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "This collection has not been created yet or is currently frozen");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L274

```solidity
        require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");
```

Could be changed to 
```solidity 
require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "This collection's data is currently frozen and cannot be updated");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L283-L284

```solidity 
            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
```

Could be changed to 

```solidity 
require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "This collection's data is currently frozen and cannot be updated");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L293
```solidity 
        require(isCollectionCreated[_collectionID] == true, "No Col");
```

Could be changed to 
```solidity 
require(isCollectionCreated[_collectionID] == true, "This collection has not been created yet");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L308
```solidity 
        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
```

Could be changed to 

```solidity 
require(block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "The set final supply time has not passed yet");
```


# [N-01] function freezeCollection needs an event emission to notify external listeners that the collection has been frozen. This could be important as it will provide/maintain transparency to users. 

There are 1 instances of this issue: 

 https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292

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

# [N-02] A More Clear Descriptive Message Can Be Written 
There are 32 instances with this issue: 

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
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L144-L145
```solidity
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```

Could be change to 

```solidity 
require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "You are not authorized to perform this action");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L151-L152

```solidity
      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
```

Could be change to 

```solidity
require(adminsContract.retrieveCollectionAdmin(msg.sender, _collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "You are not authorized to perform this action");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L158-L159
```solidity
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
```

```solidity
require(gencore.retrievewereDataAdded(_collectionID) == true, "Please add data to this collection before minting");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L171-L172
```solidity
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
```

Could be changed to 

```solidity
require(setMintingCosts[_collectionID] == true, "Please set the minting costs for this collection before minting");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L182

```solidity
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
```
could be changed to 

```solidity
require(gencore.retrievewereDataAdded(_collectionID) == true, "Please add data to this collection before minting");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L186-L187

```solidity
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
```

```solidity
require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "This collection has no remaining supply to mint");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L197-L198

```solidity
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
```

could be change to 
```solidity 
require(setMintingCosts[_collectionID] == true, "Please set the minting costs for this collection before minting");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L211-L212

```solidity
                require(isAllowedToMint == true, "No delegation");
```

Could be changed to 
```solidity
require(isAllowedToMint == true, "This collection is not currently allowed to mint tokens");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L213-L217
```solidity 
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");
```

Could be changed to 

```solidity
require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "You have exceeded the maximum allowance for this collection");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L223-L224

```solidity 
            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
```

Could be changed to 

```solidity 
require(_numberOfTokens <= gencore.viewMaxAllowance(col), "You cannot mint more tokens than the maximum allowance for this collection");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L224-L225

```solidity
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
```
Could be changed to 
```solidity
require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Exceeded maximum token allowance");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L228-L229

```solidity
revert("No Minting")
```

Could be changed to 

```solidity
revert("Minting Not Allowed");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L360

```solidity
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```

Could be changed to 

```solidity
require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "Token supply limit reached");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361-L362

```solidity
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```

Could be changed to 

```solidity 
require(msg.value >= (getPrice(col) * 1), "The amount of ETH sent is not sufficient to cover the price of the collection");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L381-L382

```solidity
        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
```

Could be changed to 
```solidity
require(collectionArtistPrimaryAddresses[_collectionID].status == false, "This collection has already been approved");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L395

```solidity
        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
```

Could be changed to 

```solidity 
require(collectionArtistSecondaryAddresses[_collectionID].status == false, "This collection has already been approved");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L416-L417
```solidity
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
```
Could be changed to 

```solidity
require(collectionArtistPrimaryAddresses[_collectionID].status == true, "The artist needs to accept royalties for this collection");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L455-L456

```solidity
    require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

Could be change to 

```solidity
require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "The provided contract is not an admin contract");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32-L33

```solidity
  require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
```

Could be changed to 

```solidity
require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "You are not authorized to perform this action. Only admins or the owner can perform this action");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L35
```solidity
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```

Could be changed to 

```solidity 
require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "You are not authorized to perform this action. Only function admins can perform this action");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L48-L49
```solidity
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```
Could be changed to 
```solidity
require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "You are not authorized to perform this action. Only function admins can perform this action");
```
https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L95-L96
```solidity
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
Could be changed to 
```solidity
require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "The provided contract is not an admin contract");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L36-L37

```solidity
        require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
```

Could be changed to 

```solidity 
require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "You are not authorized to perform this action. Only function admins can perform this action");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L62-L63

```solidity 
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

Could be changed to 

```solidity 
require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "The provided contract is not an admin contract");
```

https://vscode.dev/github/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L32

```solidity 
      require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
```

Could be changed to 

```solidity 
require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Error: You are not authorized to execute this function. Only the current highest bidder or an authorized administrator can execute this function on the contract.");
```









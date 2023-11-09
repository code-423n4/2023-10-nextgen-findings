# Code:
## ```  NextGenCore ::setCollectionData()```
### https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L155
### https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L156
## ``` NextGenCore::airDropTokens() ```
### https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L181
## ``` NextGenCore::mint() ```
### https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L192
## ``` NextGenCore::setFinalSupply() ```
### https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L310
 
# Impact:
## ```  NextGenCore ::setCollectionData()```
### _collectionID  can be 0 , If it becomes 0 then reservedMinTokensIndex  & will also become 0, which in worst case scenario could be just less than the total collection supply & reservedMaxTokensIndex can become less than totalCollectionsupply 
## ``` NextGenCore::airDropTokens() ```
### no checks are added for if collectionTotalSupply < collectionCirculationSupply, so if anyhow the circulation goes above TOtoalSupply,there would be more tokens available in the periodic sale
## ``` NextGenCore::mint() ```
### no checks are added for if collectionTotalSupply < collectionCirculationSupply so if anyhow the circulation goes above TOtoalSupply,there would be more tokens available in the periodic sale

# POC :
## ```  NextGenCore ::setCollectionData()```
```
          collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);  
           collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1; 
```
## ``` NextGenCore::airDropTokens() ```
 ```
if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
```
## ``` NextGenCore::mint() ```
``` 
collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
```
## ``` NextGenCore::setFinalSupply() ```
```
function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```

# Tools Used:
### Manual Review

# Recommendation:
## ```  NextGenCore ::setCollectionData(),setFinalSupply()```
### Use a require or revert statement for checking that _collectionID is not 0
## ``` NextGenCore ::airDropTokens(), NextGenCore ::mint()```
### Add a error check for if collectionCirculationSupply > collectionTotalSupply .
The setFinalSupply function has no overflow protection when setting `collectionAdditionalData[_collectionID].reservedMaxTokensIndex`. With a `_collectionID` sufficiently large enough, then the `_collectionID * 10000000000` will certainly overflow, causing issues. This would be a much higher criticality if it weren't for the fact that `_collectionID` would have to be so incredibly large for this vulnerability to become an issue. Nonetheless, there should be the following checks to prevent issues:
- make a check to verify the `_collectionID` is associated with a collection that currently exists 
- use a library and/or a conditional check for input size to verify that `_collectionID * 10000000000` won't exceed `type(uint256).max`

Concerns the last line of the following `setFinalSupply` function:
```
    function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```
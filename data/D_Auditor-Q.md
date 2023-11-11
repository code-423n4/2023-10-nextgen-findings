## LOW-01: Missing check in `MinterContract::setCollectionCosts()` permits the mismatch of protocol's sales model and model variables.

### Brief/Explanation
The MinterContract::setCollectionCosts() responsible for setting the costs and sales model failed to check if the timePeriod and rate is a match for the sales Option

### Impact
Sales models are characterized by various conditions. A fixed price sales model is xterized by a zero timePeriod and a zero rate. Periodic sales model is characterized by a non-zero timePeriod and a non-zero/zero rate, etc. `setCollectionCosts` also explicitly requires that the salesOption/sales model is explicitly stated. What it failed to do is to match the salesOption with the provided characteristics to see if they are a match. There are various instances where the sales option/model determines the kind of operation to be conducted in a function. A wrong salesOption for the right xters or vice versa could result in DOS for a collection.

```solidity
    function setCollectionCosts(
        uint256 _collectionID, 
        uint256 _collectionMintCost,
        uint256 _collectionEndMintCost,
        uint256 _rate,
        uint256 _timePeriod,
        uint8 _salesOption, 
        address _delAddress
    ) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
        collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
        collectionPhases[_collectionID].rate = _rate;
        collectionPhases[_collectionID].timePeriod = _timePeriod;
        collectionPhases[_collectionID].salesOption = _salesOption;
        collectionPhases[_collectionID].delAddress = _delAddress;
        setMintingCosts[_collectionID] = true;
    }
```

### Recommendation
Include a check in the function that ensures that the salesOpotion and the xters are a match

---

## LOW-02: Missing check in `MinterContract::setCollectionPhases()` permits the clash of protocol's mint timing

### Brief/Explanation
`MinterContract::setCollectionPhases()` failed to ensure that the allowlist timing doesnt clash with the public sale timing.

### Impact
The Different phases in minting is dictated by the allowlist and public mint schedule set in the `MinterContract::setCollectionPhases()`. However, the function failed to ensure that the allowlist minting comes first before the public sale and that the public sale end period doesnt clash with that of the allowlist.

```solidity
    function setCollectionPhases(
        uint256 _collectionID, 
        uint _allowlistStartTime, 
        uint _allowlistEndTime, 
        uint _publicStartTime, 
        uint _publicEndTime, 
        bytes32 _merkleRoot
    ) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
    }
```

### Recommendation
Include a check in the function that aligns the different phases properly, eliminating the possibility of a clash.
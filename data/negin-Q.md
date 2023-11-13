### Low-Risk Issues  
  
| | Issue | Instances |  
| --- | --- | --- |  
| [L‑01] | Collection freezing check is missing | 1 |  
| [L‑02] | Collection delegate address existence should be checked | 1 |  
  
Total: 2 instances over 2 issues  
  
---  
  
## Low-Risk Issues  
  
### [L‑01] **Collection freezing check is missing**  
  
The `freezeCollection()` is responsible for locking the information, data and metadata of a collection for ever.

> The collection should exist and not be frozen.

Though, if we look at the function `freezeCollection()` inside the contract NextGenCore, we can find out that there is not a check to satisfy the abovementioned check.

```Solidity  
    function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col");
        collectionFreeze[_collectionID] = true;
    }
```
As we can see, the prior freezing check is missed.  
  
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L292](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L292)  
  
### [L‑02] **Collection delegate address existence should be checked**  
  
The `setCollectionCosts()` function is called to set the variables related to the cost of a collection. We can see that this function has some checks. If we look at the docs we can find that the NFT delegation address existence check should be priorly performed.

> * @param _delAddress Refers to the smart contract address in which the NFTDelegation smart contract will check if a delegation exists.

But if we see the contract we can find that this check is missing:

```solidity  
    function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
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

*There is an instance of this issue:*  
  

  
Link(s): [https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157)  
  
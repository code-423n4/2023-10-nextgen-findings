Minting costs should be set in getPrice().
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L530.

The following code is used in getPrice():
```
 if (collectionPhases[_collectionId].rate > 0) {
            return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
            } else {
                return collectionPhases[_collectionId].collectionMintCost;
            }
```
The function should revert earlier to save gas if collection mintcost is not set:
` require(setMintingCosts[_collectionID] == true, "Set Minting Costs");`
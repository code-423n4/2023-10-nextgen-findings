# ```  NextGenCore ::setCollectionData()```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L152
 
## Impact:
Wastes gas

## Recommendation:
In solidity ,if we don't explicitly initialize a state variable (collectionCirculationSupply) to a specific value, it will be assigned the default value for its type. For integer types like uint256, the default value is 0.
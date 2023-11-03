[/hardhat/smart-contracts/MinterContract.sol:530](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L530):
Move storage variables to function variables to save gas on reading from a storage and making code cleaner:
```
uint256 salesOption = collectionPhases[_collectionId].salesOption;
uint256 rate = collectionPhases[_collectionId].rate;
uint256 mintCost = collectionPhases[_collectionId].collectionMintCost;
uint256 timePeriod = collectionPhases[_collectionId].timePeriod;
uint256 allowlistStartTime = collectionPhases[_collectionId]
         .allowlistStartTime;
uint256 endMintCost = collectionPhases[_collectionId]
         .collectionEndMintCost;
```


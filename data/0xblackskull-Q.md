There are important check is missing for input validation which can break the whole protocol

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L170
MinterContract:setCollectionPhases
```solidity
require(_allowlistStartTime < _allowlistEndTime );
require(_publicStartTime < _publicEndTime );
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L315
MinterContract:initializeExternalBurnOrSwap
```solidity
require(_tokmin < _tokmin );
```


|no |issue|number of issue|
|------|-----|---------------|
|[G-01]|mintIndex and collectionTokenMintIndex are equal in some cases.|3|


## [G01] mintIndex and collectionTokenMintIndex are equal in some cases.
In these three cases, collectionTokenMintIndex can be used instead of mintIndex to save gas costs.


```solidity

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267
```
264        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
267        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);

```solidity

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L281
```

279        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
281        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);

```solidity

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362
```
359        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
362        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
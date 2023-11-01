## The same value is assigned to multiple local variables.

There are three instances of this:
File: `smart-contracts/MinterContract.sol`

In the function `burnToMint`
```
263        uint256 collectionTokenMintIndex;
264        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
// The same value assigned to collectionTokenMintIndex in line 264 is also assigned to mintIndex in line 267
267        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
```

In the function `mintAndAuction`
```
278        uint256 collectionTokenMintIndex;
279        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
// The same value assigned to collectionTokenMintIndex in line 279 is also assigned to mintIndex in line 281
281        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```

In the function `burnOrSwapExternalToMint`
```
358        uint256 collectionTokenMintIndex;
359        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
// The same value assigned to collectionTokenMintIndex in line 359 is also assigned to mintIndex in line 362
362        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
```

In each case, I would recommend removing the two lines that declare and assign value to `collectionTokenMintIndex` and replace any use of it with `mintIndex`.

**NOTE:** The bot-report found a gas optimization `[G-18]` that overlaps with some of these lines. The bot-report recommended caching the results of the function calls, to not repeat them within the function. This finding leads to further optimization by declaring and assigning values to fewer variables.
# Risk
Gas

# Title

Avoid using nested loops

## Impact

In Solidity, nested loops can be gas-inefficient because they increase the complexity of the function, leading to higher gas costs. Each iteration of the inner loop represents additional computational work, which translates into higher gas costs.

In the `airDropTokens()` function in `MinterContract.sol` contract, there are two nested loops. The outer loop iterates over the `_recipients` array.
The inner loop iterates over the `_numberOfTokens` array.
This results in a time complexity of `O(n*m)`, where `n` is the length of the `_recipients` array and `m` is the length of the `_numberOfTokens` array. This can lead to high gas costs if these arrays are large.

`MinterContract.sol:181:`
```solidity

function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
    require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
    uint256 collectionTokenMintIndex;
@>  for (uint256 y=0; y< _recipients.length; y++) {
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
@>      for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
            gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
        }
    }
}

```
## Links

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181-L192

## Recommendation

You can optimize the function as rewrite the function to avoid nested loops. The modified function can be:

``` diff

function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
   require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
   uint256 collectionTokenMintIndex;
+  uint256 totalTokens = 0;
+  for (uint256 y=0; y< _recipients.length; y++) {
+     totalTokens += _numberOfTokens[y];
+  }
+  collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + totalTokens - 1;
+  require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
+  for(uint256 i = 0; i < totalTokens; i++) {
+      uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + i;
+      gencore.airDropTokens(mintIndex, _recipients[i % _recipients.length], _tokenData[i % _tokenData.length], _saltfun_o[i % _saltfun_o.length], _collectionID);
+  }

-   for (uint256 y=0; y< _recipients.length; y++) {
-        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
-        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
-        for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
-            uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
-            gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
-        }
-    }
}

```

In this optimized version, the function first calculates the total number of tokens to be minted, then iterates over the total number of tokens in a single loop. This reduces the time complexity from `O(n*m)` to `O(n)`, which can significantly reduce the gas costs. The modulus operator `(%)` is used to cycle through the `_recipients`, `_tokenData`, and `_saltfun_o` arrays.
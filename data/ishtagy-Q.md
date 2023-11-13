# Missing collection existence check in changeTokenData function

### Severity

Low risk

### Relevant GitHub links

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L273

## Summary:

The `changeTokenData` function does not include a check for the existence of the collection, as specified in the documentation. This can lead to unexpected behavior or vulnerabilities.

## Vulnerability Details:

`Documentation Requirements:`
The documentation specifies that the `changeTokenData` function can be called by a global admin or a function admin, and the collection should exist and not be frozen.

`Issue:`
The `changeTokenData` function does enforce checks for global admin or function admin roles and the frozen status of the collection. However, it doesn't check for the existence of the collection, as explicitly required by the documentation.

```javascript
function changeTokenData(uint256 _tokenId, string memory newData) public FunctionAdminRequired(this.changeTokenData.selector) {
        require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");
        _requireMinted(_tokenId);
        tokenData[_tokenId] = newData;
    }
```

## Impact:

The absence of a collection existence check in the changeTokenData function may lead to unexpected behavior or potential vulnerabilities, as the function may be invoked even when the collection does not exist.

## Tools Used

Manual Review

## Recommendations:

Add check for the existence of the collection

```diff
+ require(isCollectionCreated[_collectionID] == true, "No Col");
```
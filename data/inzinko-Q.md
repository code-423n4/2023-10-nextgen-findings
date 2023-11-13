## The updateImagesAndAttributes Function, assumes a certain way of arranging data 

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281-L288

This function assumes that the way tokens id are arranged in the array and the way image and attributes are matched against each other, which means it expects that any token id at a particular index should represent the same index images in the images array and the same attributes in the attributes array.

```solidity
function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
        for (uint256 x; x < _tokenId.length; x++) {
            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
            _requireMinted(_tokenId[x]);
            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
        }
    }
```

If this is not the case, tokens may find themselves with, other tokens images and attributes so it is better not to make that mistake

Mitigation is to ensure that every data entered inside this array matches the index at which they stored in the tokenId array

## [1] TITLE
Missing check for array length mismatch can lead to token corruption.

## Proof of Concept
In the ```NextgenCore.sol``` contract calling [```updateImagesAndAttributes(...)```](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281) is susceptible to data corruption because the function does not check if there is a mismatch between the length of the ```_tokenId```, ```_images``` and the ```_attributes```.

```solidity
function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
        // @audit missing check for matching token lengths can lead to data corruption
        for (uint256 x; x < _tokenId.length; x++) {
            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
            _requireMinted(_tokenId[x]);
            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
        }
    }
```

A mismatch in the length of the any of the arrays can lead to data corruption


## Recommended Mitigation Steps
add a ```require``` statement to check the length of the 3 arrays are equal


## [2] TITLE
Missing check for array length mismatch can lead to token corruption.

## Proof of Concept
In the ```MinterContract.sol``` contract calling [```airDropTokens(...)```](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181-L192) is susceptible to data corruption because the function does not check if there is a mismatch between the length of the input arguments.

```solidity
    function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
            }
        }
    }
```

A mismatch in the length of the any of the arrays can lead to data corruption


## Recommended Mitigation Steps
add a ```require``` statement to check the length of the arrays arguments are equal.
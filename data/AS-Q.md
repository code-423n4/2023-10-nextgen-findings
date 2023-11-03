# Issue 1

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L227C4-L232

If nobody of admins set a randomizer to the collection, any kind of mint will be failed without a specific reason because they call the `_mintProcessing` function.

```solidity
    function _mintProcessing(uint256 _mintIndex, address _recipient, string memory _tokenData, uint256 _collectionID, uint256 _saltfun_o) internal {
        tokenData[_mintIndex] = _tokenData;
        // @audit check randomizer is 0
        collectionAdditionalData[_collectionID].randomizer.calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
        tokenIdsToCollectionIds[_mintIndex] = _collectionID;
        _safeMint(_recipient, _mintIndex);
    }
```

# Mitigation Steps

We can add the following line to `_mintProcessing` function
```solidity
require(collectionAdditionalData[_collectionID].randomizerContract != address(0x0));
```

# Issue 2
Protocol should refund remained ETH to customer if he funded more than needed
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L238
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L271
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L364
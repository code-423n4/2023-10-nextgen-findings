# Impact
token additional metadata is still visible after token burning.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L204-L209
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L220

# Proof of Concept
There are nowhere in token burn function deleting data from mapping `tokenData`.
```
function burn(uint256 _collectionID, uint256 _tokenId) public {
        require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");
        require(
            (_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex)
                && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex),
            "id err"
        );
        _burn(_tokenId);
        burnAmount[_collectionID] = burnAmount[_collectionID] + 1;
}
```

# Recommended Mitigation Steps
add this: `delete tokenData[_tokenId];`
after: `_burn(_tokenId);`
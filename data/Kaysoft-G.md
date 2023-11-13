## [Gas-1] Assigning the input parameter to another newly created variable will cost more gas.

Creating the `tokData` variable in `MinterContract.sol#burnOrSwapExternalToMint(...) function will cost more gas. Use the `_tokenData` input parameter directly instead of creating new memory space.

File: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L344

```
function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
...      
        string memory tokData = _tokenData;
...
}
```
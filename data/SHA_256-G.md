# [G-01] : Caching the length in `for` loops saves gas
line of code : ```for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++)  ```
In the above case, the solidity compiler will always read the length of the array during each iteration. That is,

1.if it is a storage array, this is an extra sload operation (100 additional extra gas (EIP-2929 15) for each iteration except for the first),
2.if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first),
3.if it is a calldata array, this is an extra calldataload operation (3 additional gas for each iteration except for the first)
This extra costs can be avoided by caching the array length (in stack):
```
uint length = auctionInfoData[_tokenid].length;
for (uint i = 0; i < length; i++) {
    // codes
}
```
total instances :https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L69,
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87-L90,
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L104-L110,
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L134-L136

# [G‑02] Consider using `ERC721A` instead of `ERC721`
ERC721A(https://www.erc721a.org/) is an improved implementation of IERC721 with significant gas savings for minting multiple NFTs in a single transaction.
instance: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/ERC721Enumerable.sol#L15




  
## Title
No incentive for users to mint NFTs from functions `burnToMint()` and `burnOrSwapExternalToMint()`

## Impact
Low impact.
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L258-L272
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326-L365
No user would be incentivized to mint and NFT using these 2 functions because there is no benefit from using the normal mint function.

## Explanation
A normal user can mint collection NFTs in 3 ways:
1. Minting a new NFT calling `mint()` function
2. Minting a new NFT calling `burnToMint()` function and burning an NFT of an other collection
3. Minting a new NFT calling `burnOrSwapExternalToMint()` function and burning or swapping an NFT of an external collection

However, minting NFTs burning other NFTs or swapping has not benefit from minting normally. Users have to pay exactly the same amount of eth for minting new NFTs and as an extra burning their NFTs from other collections. With this approach who would use any of these functions `burnToMint()` `burnOrSwapExternalToMint()` to mint new NFTs?

## Tools Used
Manual review

## Recommended Mitigation Steps
Add a discount or any type of benefit for burning their NFTs so they can mint a collection NFT with less amount of funds.
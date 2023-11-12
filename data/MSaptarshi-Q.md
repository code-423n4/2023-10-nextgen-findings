 ## [1]_collectionID  can't be 0 , If it becomes 0 then reservedMinTokensIndex  & will also become 0, which in worst case scenario could be just less than the total collection supply & reservedMaxTokensIndex can't become less than totalCollectionsupply 
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L155
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L156
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L310
### Recommendation:
Use a require or revert statement for checking that _collectionID is not 0


## [2]no checks are added for if collectionTotalSupply < collectionCirculationSupply, so if anyhow the circulation goes above TotalSupply,there would be more tokens available in the periodic sale:
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L181 
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L192 
### Recommendation:
Add a error check for if collectionCirculationSupply > collectionTotalSupply .



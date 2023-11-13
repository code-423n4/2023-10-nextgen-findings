## Title
Uninitialized Variable Causes Function to revert in MinterContract.sol

## Impact
`MinterContract.getPrice` function will revert if timePeriod has not been set.

## Description
The code segment involves a calculation that depends on `collectionPhases[_collectionId].timePeriod`. However, there is no check to ensure that timePeriod has been set before this calculation is executed. 

## Location
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L546

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add a `require` check at the beginning of the function:
`require(setMintingCosts[_collectionID] == true, "Set Minting Costs");` 

## Issue Type
Invalid Validation
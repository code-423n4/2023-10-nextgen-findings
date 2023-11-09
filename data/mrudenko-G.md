# Gas Optimization Report

## [G-1] Structured Argument Passing
### Code
- [Line 131](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L131)
### Description
Passing a large number of arguments can be gas-intensive. Grouping arguments into a struct can reduce gas costs by minimizing the stack depth during execution.

## [G-2] Struct Memory Optimization
### Code
- [Lines 131-138](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L131-L138)
### Description
Copying a struct to memory, modifying it, and then saving it back to storage is more gas-efficient than modifying storage directly, due to the high cost of SSTORE operations.

## [G-3] Return Data as Struct
### Code
- [Line 439](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L439)
- [Line 427](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L427)
- [Line 483](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L483)
- [Line 489](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L489)
- [Line 495](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L495)
- [Line 501](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L501)
### Description
Returning multiple values from a function can be optimized by using a struct. This approach can save gas by reducing the number of state reads and writes.

## [G-4] Loop Increment Optimization
### Code
- [Lines 186-190](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L186-L190)
### Description
Calculating the `mintIndex` inside a loop can be gas-inefficient. Moving the initial calculation outside the loop and incrementing it in each iteration can save gas by reducing the number of expensive arithmetic operations.
### Suggested Code Change
```solidity
// Move the calculation of mintIndex outside the loop
uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);

// Inside the loop, after each iteration, increment mintIndex by 1
for (uint256 i = 0; i < _amount; i++) {
    // ... other code ...
    mintIndex++;
}

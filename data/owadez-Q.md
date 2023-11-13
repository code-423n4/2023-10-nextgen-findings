Use short form for addresses.
in line https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L205
we can see `if (_delegator != 0x0000000000000000000000000000000000000000)` is used but a more readable form which could be used is `if (_delegator != address(0))`
This would improve readability and reduce technical debt.
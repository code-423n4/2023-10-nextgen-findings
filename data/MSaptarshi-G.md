https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L152
# [1]In solidity ,if we don't explicitly initialize a state variable to 0, it will be assigned the default value for its type. For integer types like uint256, the default value is 0.


# [2] Add a limit for the maximum number of characters per line
The solidity documentation recommends a maximum of 120 characters.
Consider adding a limit of 120 characters or less to prevent large lines.
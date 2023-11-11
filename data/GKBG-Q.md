## Impact
The state variable `minterContract` in `NextGenCore` is not set in the constructor hence upon deployment its value will be `address(0)`. A value to this variable can later be assigned with the function `addMinterContract` which only the function admin can call. However there are certain functions in this contract which are callable only by the `minterContract`. They are listed below:
* `airDropTokens`
* `mint`
* `burnToMint`

In the event a `minterContract` is not set or the function admin forgets to set it for some period of time, these functions are not callable.

The overall Impact is that the main functionalitiy of the contract can remain uncallable until `minterContract` is set.

## Proof of Concept
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178;
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L189;
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L213;

## Tools Used
Manual Review

## Recommended Mitigation Steps
Set the `minterContract` upod deployment in the `constructor`
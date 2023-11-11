https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L45-L47

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerRNG.sol#L62

The updateAdminsContract function inside NextGenRandomizerNXT does not check if the new admin contract returns true on calling the isAdminContract function, as all the other updateAdminContract do. Therefore, the provided address is not checked for a zero value, or accidentally providing the wrong address. As this is a one-step process for the most sensible update in the system, this check should be implemented.
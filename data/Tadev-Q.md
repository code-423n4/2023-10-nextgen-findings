## [L‑01] Randomizer contracts are able to update the NextGenCore contract address while documentation says Core contract is all on-chain, never changes, and admins cannot update its address

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L49

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerRNG.sol#L66

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerVRF.sol#L99

The documentation clearly says :

"While the Core contract is all on-chain and never changes, admins can update the smart contract addresses of the Minter, Admin and Randomizer contracts to provide new features."

This means that there is no situation in which the Core contract is updated, deployed to a new address, and then existent randomizers stay the same but are updated by an admin to change `gencoreContract` value.
Morever, being able to update `gencoreContract` value in the specification context generates a risk of making the randomizer contract uncallable by the real Core contract in `_mintProcessing()` function (the call will revert with `gencoreContract.setTokenHash()` call in the callback of the randomizer.

`updateCoreContract()` function should be removed from the 3 randomizer contracts to follow the specification.

## [L‑02]

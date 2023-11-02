## [L‑01] Randomizer contracts are able to update the NextGenCore contract address while documentation says Core contract is all on-chain, never changes, and admins cannot update its address

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L49

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerRNG.sol#L66

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerVRF.sol#L99

The documentation clearly says :

"While the Core contract is all on-chain and never changes, admins can update the smart contract addresses of the Minter, Admin and Randomizer contracts to provide new features."

This means that there is no situation in which the Core contract is updated, deployed to a new address, and then existent randomizers stay the same but are updated by an admin to change `gencoreContract` value.
Morever, being able to update `gencoreContract` value in the specification context generates a risk of making the randomizer contract uncallable by the real Core contract in `_mintProcessing()` function (the call will revert with `gencoreContract.setTokenHash()` call in the callback of the randomizer.

`updateCoreContract()` function should be removed from the 3 randomizer contracts to follow the specification.


## [L‑02] It is not clear how RandomizerRNG contract will be funded in order to be able to call `requestRandomWords()` function

RandomizerRNG contract, contrary to other randomizers, needs to send ETH when calling `arrngController.requestRandomWords()` function, in order to request and then obtain random numbers to generate the unique hash of the token.

Currently, there are 2 payable functions : `requestRandomWords()` and `receive()` . 
`requestRandomWords()` is problematic as the only way to call it is through gencore by calling `calculateTokenHash()` function, which is not payable. Therefore, it is only possible to send ether to this contract is to ''raw'' send ether, calling `receive()` function.

In this configuration, there is no transparency in how the contract will be funded, by who, and users cannot be sure that the randomizer will always work, as it depends on its funding which is not programmatically determined.

## [L‑03]


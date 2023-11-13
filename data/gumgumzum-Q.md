### Upgrades are not really applicable
This is mainly because :
* Upgrading the a contract needs to be done everywhere it is required at once
* Cross dependencies between contracts and the previous storage being lost

A practical example is `NextGenCore` in [NextGenMinterContract](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L448-L450), [NextGenRandomizerNXT](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L49-L52), [NextGenRandomizerRNG](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L66-L69), [NextGenRandomizerVRF](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L99-L102) :
* The `NextGenMinterContract` will have information related to the old `NextGenCore` collections unless data is migrated to the new `NextGenCore`
* You can't really update it on the randomizers as it would break the old `NextGenCore` hash calculation requests

In those conditions, an upgrade will almost always require a hole new deployment of all the contracts.

A solution for this would be to use one of the commonly known upgradable contracts patterns.

### Updating images and attributes needs to be [done](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281-L288) for each NFT

### When in use, XRandoms will returns the same `randomNumber` and `randomWord` for each NFT minted in the same block

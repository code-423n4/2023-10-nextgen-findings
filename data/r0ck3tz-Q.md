# Summary

| Id     | Title                                                                           |
|--------|---------------------------------------------------------------------------------|
| [L-01] | Ether dust accumulation in `NextGenMinterContract`                                               |
| [L-02] | It is possible to set collection data twice through `setCollectionData`                    |
| [L-03] | Weak source of randomness                    |
| [L-04] | Missing check for minting that number of tokens is not equal to 0                   |
| [L-05] | Missing input validation for `createCollection`                   |
| [N-01] | Minting cannot start until randomizer is set                   |
| [N-02] | Validate if the new contract address is correct                   |
| [N-03] | Functions can be pure                   |
| [N-04] | Function should be private                   |
| [N-05] | Mixed use of `msg.sender` and `_msgSender()`                   |

## Low Severity

### [L-01] Ether dust accumulation in `NextGenMinterContract`

The `payArtist` function divides royalties between artists and teams. The calculations are performed by multiplying the royalty by the percentage and then dividing by 100 for all payments. The issue is that the payment calculation does not take into account decimals, which can lead to ether dust accumulation with the last payment.
```solidity
artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
teamRoyalties1 = royalties * _teamperc1 / 100;
teamRoyalties2 = royalties * _teamperc2 / 100;
```
It is recommended to calculate the amount of the last payment as the difference between the value of royalties and the amount that has already been paid.

### [L-02] It is possible to set collection data twice through `setCollectionData`

The `setCollectionData` of `NextGenCore` contract does not validate that the passed `_collectionTotalSupply` is not equal to `0`. This leads to situation where it is possible to populate `collectionAdditionalData` structure and then change that data later on. In addition the `reservedMaxTokenIndex` is set to the value that belongs to the previous collection. Its because of the calculation:
```solidity
collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
```
Which will end up with `(_collectionID * 10000000000) + 0  - 1;`

### [L-03] Weak source of randomness

Contract `RandomizerNXT` and `randomPool` are using weak source of randomness which is based on the `keccak256` of multiple predictable values such as `block.prevrandao`, `blockhash` of the previous block and the `block.timestamp`.
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L57
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L36
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L41

### [L-04] Missing check for minting that number of tokens is not equal to `0`

The `mint` function of `MinterContract` allows passing the number of tokens to mint as `0`. Although this will accept the ether since the function is payable, it will not mint any NFT. It is recommended to add a check to ensure the number of tokens to mint is not equal to `0`.

### [L-05] Missing input validation for `createCollection`

The `NextGenCore` contract implements the `createCollection` function that accepts multiple strings, but it lacks any type of validation, such as checking if the strings are not empty. It is recommended to add validation to the createCollection function.

## Notes

### [N-01] Minting cannot start until randomizer is set

The protocol requires the `randomizer` for the collection to be set, otherwise, it will revert at a call to calculate the hash. It is recommended to disallow any minting if the randomizer is not set.

### [N-02] Validate if the new contract address is correct

There are multiple update contract address functions that do not check if the new address is the correct contract.
- `updateAdminsContract` of `RandomizerNXT` does not check if the new admin contract is the actual admin contract.
- `updateCoreContract` of `NextGenRandomizerNXT` does not check if the new core contract is the actual core contract.
- `updateCoreContract` of `NextGenRandomizerRNG` does not check if the new core contract is the actual core contract.
 It is recommended to add additional check to update contract address functions to ensure new address is an address of the correct contract.

### [N-03] Functions can be pure

The `isRandomizerContract` funtion of `NextGenRandomizerNXT`, `NextGenRandomizerRNG` `NextGenRandomizerVRF` contracts are view but can be set as pure.

### [N-04] Function should be set as private

- The `requestRandomWords` function in `NextGenRandomizerRNG` should be private since it is only called by `calculateTokenHash`. In addition it will be possible to remove redundant check for `msg.sender == gencore`
- The `requestRandomWords` function in `NextGenRandomizerVRF` should be private since it is only called by `calculateTokenHash`. In addition it will be possible to remove redundant check for `msg.sender == gencore`

### [N-05] Mixed use of `msg.sender` and `_msgSender()`

The `NextGenAdmins` contract is using `msg.sender` and `_msgSender()` values at the same time. Consider either using everywhere `msg.sender` or in case meta transactions are expected to be supported use `_msgSender()`.

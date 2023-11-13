# Unnecessary variable update in `proposePrimaryAddressesAndPercentages()` function

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L389](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L389)

In the contract `MinterContract.sol`, the function `proposePrimaryAddressesAndPercentages()` first verifies that `collectionArtistPrimaryAddresses[_collectionID].status` is `false`.

Later on, it sets the status to `false` while it is already at `false`
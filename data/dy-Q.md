## Issues across multiple contracts

1. No functionality from `Ownable` is used in `NextGenMinterContract`, `NextGenRandomizerRNG` or `NextGenRandomizerVRF` and the owners of these contracts do not have any special rights. Therefore, the inheritance can be removed to save on contract size.
2. In some areas of the code, a contract and its address are stored in separate state variables. To reduce storage and code size, the address variable could be removed. When needed, the address of a contract object can be retrieved through casting, e.g. `address(contractInstance)`
2.1. `NextGenCore.collectionAdditionalDataStructure` contains the variables `address randomizerContract` and `IRandomizer randomizer`. The former could be removed.
2.2. `NextGenRandomizerNXT` contains the state variables `address gencore` and `INextGenCore gencoreContract`. The former could be removed.
3. Multiple contracts contain the same admin access modifiers. These modifiers could be placed in a parent contract, e.g. `Adminable`, which could then be inherited by `NextGenCore`, `NextGenMinterContract`, `NextGenRandomizerNXT`, `NextGenRandomizerRNG` and `NextGenRandomizerVRF`.

## `NextGenCore`

1. [NextGenCore.sol#110](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110): `newCollectionIndex` can be set to 1 directly instead of incremented from 0.
2. [NextGenCore.sol#111](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L111): The `_setDefaultRoyalty` function call could take a constructor argument for the receiver address, rather than the address being hardcoded. This would make the contract more flexible and avoid the necessity to change the default royalty address after deployment, should the current hardcoded address be deprecated in future.
3. [NextGenCore.sol#257](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L257): The function `artistSignature` could include a require statement ensuring that `_signature` has a length greater than 0, to prevent artists from signing with empty strings.
4. [NextGenCore.sol#101](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L101), [NextGenCore:#158](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L158), [NextGenCore.sol#259](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L259): If the above suggestion is implemented, the state variable `artistSigned` could be removed, and checks for `artistSigned[_collectionID] == false` could be replaced with checks for `artistsSignature[_collectionID].length > 0`.

### `NextGenMinterContract`

1. [MinterContract.sol#359-362](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L359-L362): A duplicated variable is present in `burnToMint` -- `collectionTokenMintIndex` stores the same value as `mintIndex` below it. `mintIndex` can be removed and replaced with `collectionTokenMintIndex` in the call to `gencore.burnToMint`.
2. [MinterContrac.solt#296](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L296): A `require` statement could be added to ensure that `_auctionEndTime` is in the future, i.e. `> block.timestamp`. Optionally, a minimum auction timespan could be enforced here. This would prevent tokens from being locked in too-short auctions.
3. [MinterContract.sol#320,321](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L320-L321): A `require` could be added to the `initializeExternalBurnOrSwap` to ensure `_tokmax > _tokmin`.
4. [MinterContract.sol#361](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L361): `* 1` is unnecessary and can be removed.
5. [MinterContract.sol#369](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L369): A `require` statement could be added to the beginning of `setPrimaryAndSecondarySplits` to ensure that `collectionPrimaryAddresses[_collectionID] == false`. This would prevent splits and addresses/percentages from being configured out of order and moving out of sync. !!! TODO maybe expand this
6. [MinterContract.sol#373](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L373): Instead of storing the team's primary and secondary splits, these could be calculated when needed by subtracting the artist's split from 100.
7.  [MinterContract.sol#417](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L417) The word "grater" in the `require` statement should be spelt "greater".

### `XRandoms`

[XRandoms#18](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L18): `wordsList` could be made into a private state variable rather than being recreated and loaded into memory every time `getWord` is called. The contents of the word list would be visible to a sufficiently technical user in either case.
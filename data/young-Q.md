- `newCollectionIndex` is not Starting from `0`

In the contract `NextGenCore.sol`, the `newCollectionIndex` starts from `1` which means that the `0`th collection will never be created. 

- `onchainMetadata` consistency is not guaranteed

`onchainMetadata` is used to define if the metadata is fetched from the internet(via baseuri) or from the contract. When the `status` is switched, the new meta info may not be the same.
    
- struct `collectionSecondaryAddresses` is redundant

The struct `collectionPrimaryAddresses` and `collectionSecondaryAddresses` are actually the same. So it's redundant to define a different struct name and attribute name.

- redundant phrase `status = false`

The `collectionArtistPrimaryAddresses[_collectionID].status = false;` is redundant as it will never change anything.

- Lack of Function to rescue/sweep tokens/ETH

The MinterContract deals with user funds, but there is no rescue/sweep function. Any funds that are sent to the contract are forever locked and lost.

- Lack of checking array length equivalence

In the `airDropTokens` function in contract `MinterContract`, the length of `_recipient` and `_tokenData` is not checked. If the length is not equal, the transaction may revert under this situation.
    
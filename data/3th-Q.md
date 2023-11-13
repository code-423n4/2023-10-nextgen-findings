##### [L-01] Off-by-one error in `NextGenCore` allows for one extra token to be minted beyond the defined `totalSupply`

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L181
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L192
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L217

The final check before minting in `NextGenCore` (L181) checks whether the `totalSupply` is greater than or equal to the `collectionCirculatingSupply`, which means that when they are equal — and the circulating supply has reached the defined capacity — the condition is satisfied and another token will be minted. 

This is low risk because the same check is actually implemented correctly in each of `NextGenMinterContract`'s entry points to this function. It is still worth fixing, however, because future external calls to `mint` may miss it.

This should be changed to:

```solidity
if (collectionAdditionalData[_collectionID].collectionTotalSupply > collectionAdditionalData[_collectionID].collectionCirculationSupply) { ... }

```


##### [L-02] `collectionCirculationSupply` can increment without minting a token

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L180
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L191
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L216

In `core.mint` and `core.burnToMint`, the `collectionCirculationSupply` is incremented before the conditional logic on the next line checks whether enough supply still remains to mint a token. If enough supply does not exist, the `mint` does not occur, but the transaction still succeeds — which means the `collectionCirculationSupply` now includes one token that was never actually minted.

This is low risk, since there is currently a redundant check for the supply in all of the user-facing functions that call these vulnerable ones. It is worth fixing anyway, though, since this check could easily be forgotten or removed in the course of development.


##### [L-03] Users can re-enter `burnToMint` after `mint` but before `burn`

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L218

Since `burnToMint` fails to follow the "checks, effects, interactions" pattern, an attacker can re-enter `burnToMint` from the `onERC721Received` callback in a malicious contract before their token has been burned. This is low risk, since the transaction will still fail if the attacker attempts to mint more than one token with only one of the "mint passes." But it should still be fixed urgently, since it is extremely possible that an exploitable opportunity would emerge here in the future.


##### [L-04] When a collection is configured to use `RandomizerNXT`, new mints can be previewed and conditionally reattempted in `onERC721Received`

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L58

Since `RandomizerNXT` has already set the `tokenHash` by the time `Core` calls `_safeMint`, users could potentially preview some data about their mint in `onERC721Received` before the transaction has completed, then `revert` and retry if desired. This is low risk, since this would all need to happen programmatically, and metadata and rarity-related attributes should not be available at this point in the minting process. It is certainly not beyond imagination that some collections might one day set data at the time of minting that *can* be advantageous to know in advance, though, so it should be patched regardless.
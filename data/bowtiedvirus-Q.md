# [L1] Reentrancy is possible in MintContract:mint() and MintContract:burnToMint()

Note: This is the same issue as [L5 from the bot report](https://gist.github.com/code423n4/b979a00ee3ea3f7d36ee39e2a536ba8a#l-05-functions-calling-contracts-with-transfer-hooks-are-missing-reentrancy-guards), but the report did not catch all instances of this issue.

## Summary
An argument, `address _mintTo`, passed to `MinterContract:mint()` opens up the possibility of reentrancy. `_mintTo` is eventually passed to `ERC721:_safeMint()`, which calls `IERC721Receiver(to).onERC721Received` on the target contract. This `to` contract can then, reenter `MintContract:mint()` before the first call finishes. This is similar to `MintContract:burnOrSwapExternalToMint()` too.

This is also true for `MintContract:burnToMint()`, except that the recipient address is not passed in by the user. The owner of a token is used, but that could also be a malicious contract.

While not posing an obvious risk due to most effects completing before the possible reentrancy, this still leaves an open door to this style of attack.

Instances not found by the automated report:

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L231
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326


## Recommendation
Similar to the recommended from the automated finding, add a reentrancy guard modifier to any functions that have this interaction and are user-facing.

# [L2] Using block based inputs to generate randomness is open to manipulation

**Effected Contract:** RandomizerNXT.sol, XRandoms.sol

## Summary
Using block.number and block.timestamp as a source of randomness is commonly advised against, as the outcome can be manipulated by callers and miners.

Instance:
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L35-L43

## Recommendation
There are other VRF contracts in-scope already. I recommend only using those for production, otherwise randomness can be gamed to get optimal token hashes.
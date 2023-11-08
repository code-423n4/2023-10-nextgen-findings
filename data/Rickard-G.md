## [G-01] Use indexed events for value types as they are less costly compared to non-indexed ones
## Relevant GitHub Links
[https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerRNG.sol#L24](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerRNG.sol#L24)

[https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerVRF.sol#L20](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerVRF.sol#L20)
## Summary
## Vulnerability Details
Using the `indexed` keyword for [value types](https://docs.soliditylang.org/en/v0.8.21/types.html#value-types) (`bool/int/address/string/bytes`) saves gas costs, as seen in [this example](https://gist.github.com/0xxfu/c292a65ecb61cae6fd2090366ea0877e).

However, this is only the case for value types, whereas indexing [reference types](https://docs.soliditylang.org/en/v0.8.21/types.html#reference-types) (`array/struct`) are more expensive than their unindexed version.

There are `2` instances of this issue:

- The following variables should be indexed in [NextGenRandomizerRNG.Withdraw(address, bool, uint256)](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerRNG.sol#L24):
    - status
- The following variables should be indexed in [NextGenRandomizerVRF.RequestFulfilled(uint256, uint256[])](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerVRF.sol#L20):
    - requestId
    - randomWords

## Impact
## Tools Used
## Recommendations
Using the `indexed` keyword for values types `bool/int/address/string/bytes` in event.
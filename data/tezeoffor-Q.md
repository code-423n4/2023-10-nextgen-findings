Use of blockhash, block.difficulty and other fields is also insecure, as they're controlled by the miner.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L54-L59

**Reccomendation**
Use external sources of randomness via oracles, and cryptographically checking the outcome of the oracle on-chain. e.g. use Chainlink VRF.
## [QA1]  Limited Entropy and Public Information Usage Impact Hash Uniqueness
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L55-L58
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L35-L43

The use of modular arithmetic to limit random numbers (% 1000) significantly diminishes the entropy, making the random values more predictable and susceptible to brute-force attacks
```
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
```
Similarly, the reliance on a finite 1000-word list for words reduces the overall entropy of generated hashes

Incorporating public information, such as block timestamps or previous block hashes, introduces predictability into the hash generation process. This reliance on publicly available data undermines the uniqueness of the generated hashes, creating potential vulnerabilities like collisions or preimage attacks
```
        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
```
### recomendations
Implement Enhanced Entropy Measures
Expand Wordlist for Greater Diversity
Reduce Dependency on Public Information

## [QA-2] Precision Loss in Multiplication after Division
Two instances in the codebase exhibit potential precision loss resulting from multiplication after division. In both cases, the issue arises from performing operations on integers, leading to truncation of fractional parts and a subsequent loss of precision.
```
return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));

```
```
decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
```
In the context of multiplication after division, the precision loss can be attributed to the fact that division introduces errors due to the inherent limitations of representing real numbers in binary format. When the result of a division is multiplied, these errors are amplified, leading to a loss of precision.
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L530
#### Recommendation
Rearrange the sequence of operations to prioritize multiplication before division, ensuring that fractional parts are preserved during calculations.

## [QA-3] Misleading comments
```
uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
///@audit misleading comment
// users are able to mint after a day passes
require(tDiff >= 1, "1 mint/period");
```
The highlighted code calculates the time difference (tDiff) between the current timestamp (block.timestamp) and the time of the last minting operation (timeOfLastMint). The calculated time difference is then divided by the specified time period (collectionPhases[_collectionID].timePeriod). The subsequent comment suggests that users are allowed to mint once a day, and there is a requirement check to ensure this condition.

Misleading Comment Explanation:
The comment ```// users are able to mint after a day passes``` may be misleading because the actual requirement check `require(tDiff >= 1, "1 mint/period");` checks if at least one time period has passed since the last mint. It doesn't explicitly enforce a one-day waiting period.
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L248-L254
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L291-L294

## [QA-4] Lack of Duplicate Address Checking in `airDropTokens` Function
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181
The airDropTokens function  fails to incorporate checks for possible duplicate addresses within the provided _recipients array. This omission may lead to unintended consequences such as tokens being airdropped to the same address multiple times during a single call.

Implement a check for duplicate addresses within the _recipients array before initiating the airdrop.

## [QA-5]  Uninitialized Minter Contract Vulnerability in NextGen Contract.
The  `NextGen contract` exhibits a vulnerability related to the uninitialized state of the minter contract during deployment. This issue arises because certain functions, including the `mint` function, require the caller to be the minter contract. If the minter contract is not set during deployment and is only initialized later through a separate function, there is a window of opportunity for any user to call these functions before the minter contract is set
The vulnerability allows users to call functions with minter-only access before the minter contract is properly initialized. This could lead to unauthorized access to minting and other privileged operations, potentially resulting in unintended token creation
--Recommendation:
Initialize the Minter Contract during construction. 
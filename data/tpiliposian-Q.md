# 1. Wrong random number generator

## Description

The `XRandoms.sol` contract is used to generate random word from a given 100-length words array. But the function `getWord()` returns this:

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L28-L33

```solidity
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
```

This function is called from `randomWord()` function which can generate a random number from 0 to 99. This means that in case a random number is 0 or 1, it will take the first word from the array, and there is no chance of having a last word from the array.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L40-L43

## Remediation

Delete `else` block from the `getWord()` function, or generate a random number from 0 to 100:

```diff
--        if (id==0) {
            return wordsList[id];
--        } else {
--            return wordsList[id - 1];
--        }
```

```diff
-- uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
++ uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 101;
```

# 2. Missing boundaries on returnIndex function

## Description

In the `XRandoms.sol` contract the function `returnIndex()` is used to return an index for a given parameter `id` but there are no checks if the `id` is not greater than 99 or 100 (depending on the fix to the previous `Wrong random number generator` issue).

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L45-L47

```solidity
    function returnIndex(uint256 id) public view returns (string memory) {
        return getWord(id);
    }
```

## Remediation

Add missing checks for the `id` parameter, as written in the descriptions.
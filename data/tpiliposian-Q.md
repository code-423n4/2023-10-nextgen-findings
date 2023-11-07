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

# 3. Missing zero address checks on setter function

## Description

In the `RandomizerRNG.sol` contract, the `updateCoreContract` function is used to update the `_gencore` parameter. However, this function does not include a check to ensure that the provided `_gencore` address is not the zero address (address(0)). Allowing the zero address as a valid parameter can lead to unexpected behavior and potential vulnerabilities.

```solidity
    function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
        gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
    }
```

## Remediation

A zero address check should be added at the beginning of the `updateCoreContract` function.

# 4. Missing boundaries on RNG cost

## Description

In the `RandomizerRNG.sol` contract, the `updateRNGCost` function is used to update the `ethRequired` parameter without any bounds or limits. This means that anyone with the appropriate permissions can set the `ethRequired` value to any arbitrary number, which could potentially lead to unintended behavior or economic vulnerabilities.

```solidity
    function updateRNGCost(uint256 _ethRequired) public FunctionAdminRequired(this.updateRNGCost.selector) {
        ethRequired = _ethRequired;
    }
```

## Remediation

Add appropriate limits or constraints on the values that can be set for the `ethRequired` parameter within the `updateRNGCost` function. These limits should be defined based on the requirements and constraints of the contract.
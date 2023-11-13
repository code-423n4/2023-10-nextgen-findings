# Report

## Summary

### Non Critical Issues

Total of **2** issues:

|                                             ID                                              |                                       Issue                                        |
| :-----------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------: |
|                           [NC-1](#nc-1-misleading-function-name)                            |                              Misleading function name                              |
| [NC-2](#using-block-information-as-a-source-of-randomness-is-generally-considered-insecure) | Using block information as a source of randomness is generally considered insecure |

## Low Issues

## Non Critical Issues

### <a name="NC-1"></a>[NC-1] Misleading function name

Function XRandom#returnIndex() has misleading name - instead of an index, it actually returns a word.

```solidity
File: smart-contracts/XRandom.sol

function returnIndex(uint256 id) public view returns (string memory) {
    return getWord(id);
}
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L45-L47

### <a name="NC-2"></a>[NC-2] Using block information as a source of randomness is generally considered insecure

Using block information as a source of randomness is generally not considered secure for applications that require certain level of unpredictability. The input of the 'keccak256' function is deterministic, based on the block information. Anyone with the knowledge of the block data can potentially predict the output.

Occurrences: 2

```solidity
File: smart-contracts/XRandom.sol

function randomNumber() public view returns (uint256){
    uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
    return randomNum;
}
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L35-L38

```solidity
File: smart-contracts/XRandom.sol

function randomWord() public view returns (string memory) {
    uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
    return getWord(randomNum);
}
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L40-L43

## [01] In randomPool's wordsList array, last fruit name will never be used for randomness.

In [randomPool.sol getWord()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L18)

In the below code, The modulo is 100, so the randumNum ranges from 0 to 99, but never 100. So this calls the array of those 100 random fruit names, which gives same word for id 0 & 1. But id 100 will never be reached.

```solidity
    function randomWord() public view returns (string memory) {
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
        return getWord(randomNum);
    }
```


#### Risk

No Risk. But no usage of the last element of random word.

#### Mitigation

```diff
    function getWord(uint256 id) private pure returns (string memory) {
            string[100] memory wordsList = ["Acai", "Ackee", "Apple", ...., "Ugli", "Velvet Apple", "Watermelon"];

-           if (id==0) {
                return wordsList[id];
-           } else {
-               return wordsList[id - 1];
-           }
    }
```
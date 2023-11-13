## [01] In randomPool's wordsList array, last fruit name will never be used for randomness.

In [randomPool.sol getWord()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L18)

In the below code, The modulo is 100, so the randumNum ranges from 0 to 99, but never 100. So this calls the array of those 100 random fruit names, which gives same word for id 0 & 1. But id 100 will never be reached.

```solidity
    function randomWord() public view returns (string memory) {
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
        return getWord(randomNum);
    }
```


 Mitigation

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

## [02] Do not use `indexed` on integers in events

- using the indexed keyword for integers will make th tracking the values of that integer hard.
- When indexed is used, it will hash to filter that data easily in logs bloom filter.


Mitigation

```diff
- event PayArtist(address indexed _add, bool status, uint256 indexed funds);
- event PayTeam(address indexed _add, bool status, uint256 indexed funds);
- event Withdraw(address indexed _add, bool status, uint256 indexed funds);
+ event PayArtist(address indexed _add, bool status, uint256 funds);
+ event PayTeam(address indexed _add, bool status, uint256 funds);
+ event Withdraw(address indexed _add, bool status, uint256 funds);
```



    
## [03] Ownable is inherited, but never implemented the role.

- The core and minter contracts inherit the Ownable functions, but onlyOwner modifier is not implemented on any function. 
- And owner state variable is also not read anywhere. So Remve it to save gas on deployment. 
- Or if there was any forgotten implementation, implement it.


Risk

- No Risk.

Mitigation

```diff
- contract NextGenMinterContract is Ownable {}
+ contract NextGenMinterContract {}
```

```diff
- contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {}
+ contract NextGenCore is ERC721Enumerable, ERC2981 {}
```

## [04] `tokensAirdropPerAddress` state is updated for every airdrop, but never read or validated anywhere

- `In GenCore.airDropTokens` the state `tokensAirdropPerAddress` is updated.
- But it is no where validated for max aidrop per address or no events emitted.
- Either some validation/implementation is missing.


Mitigation
- Remove the state and the updating code block to save gas.
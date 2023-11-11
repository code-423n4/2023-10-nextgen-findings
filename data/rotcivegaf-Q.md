# [L-01] Reduce entropy in function `getWord`, same value for different id

The function `getWord` of the contract **randomPool** should returns a word from `wordsList` based on index, but instead of it returns the index sub one and if `id` is `0` returns the id `0`

```solidity
File: smart-contracts/XRandoms.sol

27:        // returns a word based on index
28:        if (id==0) {
29:            return wordsList[id];
30:        } else {
31:            return wordsList[id - 1];
32:        }
```

As we can see in this `if/else` the return for the `id` `0` and `1` it's the same value `"Acai"`

In addition, based in this comment `// returns a word based on index` and in common sense the function should return the index

## Proof of Concept

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";

import {randomPool} from "../smart-contracts/XRandoms.sol";

contract PocTest is Test {
    randomPool _randomPool;

    function setUp() public {
        _randomPool = new randomPool();
    }

    function testShouldNotBeEQ() public {
        string memory word0 = _randomPool.returnIndex(0);
        string memory word1 = _randomPool.returnIndex(1);

        console.log("Word index 0:", word0);
        console.log("Word index 1:", word1);

        assertNotEq(word0, word1, "Should not be equal");
    }

    function testShouldReturnTheWorldOfTheIndex() public {
        string[100] memory wordsList = [
            "Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange",
            "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine",
            "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian",
            "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum",
            "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango",
            "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple",
            "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma",
            "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"
        ];

        for (uint256 i; i < 100; i++) {
            assertEq(_randomPool.returnIndex(i), wordsList[i], "Should be equal");
        }
    }
}
```

## Recommended Mitigation Steps

```diff
@@ -24,13 +24,8 @@ contract randomPool {
         "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma",
         "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];

-        // returns a word based on index
-        if (id==0) {
-            return wordsList[id];
-        } else {
-            return wordsList[id - 1];
-        }
-        }
+        return wordsList[id];
+    }
```

# [L-02] Reduce entropy in function `getWord`, the id 100(`"Watermelon"`) is not used

The function `getWord` of the contract **randomPool** don't se the last index of the `wordsList` array

```solidity
File: smart-contracts/XRandoms.sol

40:    function randomWord() public view returns (string memory) {
41:        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
42:        return getWord(randomNum);
43:    }
```

As we can see the function the `randomNum` is a value between `0` and `99` and function `getWord` expects `100` as maximum id, for this the word `Watermelon`(id: `100`) is unachievable

## Recommended Mitigation Steps

Look [L-01]

# [L-03] Weak random on `randomNumber()` and `randomWord()` functions of **randomPool**

On blockchain don't exist the randoms values, the result of this both functions can be precalculated, manipulated the result. With the help of a [Flashbots](https://www.flashbots.net/) a wallet can send the transaction only if the result it's seems favorable for it

```solidity
File: smart-contracts/XRandoms.sol

35:    function randomNumber() public view returns (uint256){
36:        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
37:        return randomNum;
38:    }

40:    function randomWord() public view returns (string memory) {
41:        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
42:        return getWord(randomNum);
43:    }
```

# [L-04] Goerli's testnet `keyHash` on **NextGenRandomizerVRF** contract

The `keyHash` used in **NextGenRandomizerVRF** contract belongs to Goerli testnet: https://docs.chain.link/vrf/v2/subscription/supported-networks/#goerli-testnet

The contract should use the Ethereum mainnet one in this case: https://docs.chain.link/vrf/v2/subscription/supported-networks/#ethereum-mainnet

```solidity
File: smart-contracts/RandomizerVRF.sol

26:    bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;
```

# [L-05] The `_auctionEndTime` can be an old date

The `_auctionEndTime` it does not check that the date is greater than the current one

- File: smart-contracts/MinterContract.sol

```solidity
    // mint and auction

    function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
        uint timeOfLastMint;
        // check 1 per period
        if (lastMintDate[_collectionID] == 0) {
        // for public sale set the allowlist the same time as publicsale
            timeOfLastMint = collectionPhases[_collectionID].allowlistStartTime - collectionPhases[_collectionID].timePeriod;
        } else {
            timeOfLastMint =  lastMintDate[_collectionID];
        }
        // uint calculates if period has passed in order to allow minting
        uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
        // users are able to mint after a day passes
        require(tDiff>=1, "1 mint/period");
        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
        mintToAuctionData[mintIndex] = _auctionEndTime;
        mintToAuctionStatus[mintIndex] = true;
    }
```
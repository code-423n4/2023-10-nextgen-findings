In smart-contracts/XRandoms.sol, wordsList is string memory type in getWord() function. This make cost when calling getWord().

```
    function getWord(uint256 id) private pure returns (string memory) {
        
        // array storing the words list
        string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", ...

        // returns a word based on index
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }
```

So It could be improved by using storage access.

Here is full code of XRandoms.sol changed.

```
// SPDX-License-Identifier: MIT

/**
 *
 *  @title: NextGen Word Pool
 *  @date: 09-October-2023 
 *  @version: 1.1
 *  @author: 6529 team
 */

pragma solidity ^0.8.19;

contract randomPool {
    // array storing the words list
    string[100] public wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", 
    "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine", 
    "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian", 
    "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum", 
    "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango", 
    "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple", 
    "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma", 
    "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];


    function getWord(uint256 id) private view returns (string storage) {
        // returns a word based on index
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }

    function randomNumber() public view returns (uint256){
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
        return randomNum;
    }

    function randomWord() public view returns (string memory) {
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
        return getWord(randomNum);
    }

    function returnIndex(uint256 id) public view returns (string memory) {
        return getWord(id);
    }

}
```
Here is my test code to test Contract, randomPool.t.sol

```
pragma solidity ^0.8.19;

import "forge-std/Test.sol";
import "forge-std/console.sol";

import "../src/XRandoms.sol";

contract RandomPoolTest is Test {
    randomPool pool;

    function setUp() public {
        pool = new randomPool();
    }

    function testRandomNumber() public {
        uint256 randomNum = pool.randomNumber();
        assertTrue(randomNum >= 0 && randomNum < 1000);
    }

    function testRandomWord() public {
        string memory word = pool.randomWord();
        assertTrue(bytes(word).length > 0);
    }

    function testReturnIndex() public {
        string memory word = pool.returnIndex(1);
        assertEq(word, "Acai");
    }

    function testFailReturnIndexOutOfBounds() public {
        pool.returnIndex(101); // This should fail because the index is out of bounds
    }
}
```

Here is gas report before patched.

```
$ forge test --contracts test/randomPool.t.sol --gas-report
...
Running 4 tests for test/randomPool.t.sol:RandomPoolTest
[PASS] testFailReturnIndexOutOfBounds() (gas: 13329)
[PASS] testRandomNumber() (gas: 5787)
[PASS] testRandomWord() (gas: 14637)
[PASS] testReturnIndex() (gas: 15001)
Test result: ok. 4 passed; 0 failed; 0 skipped; finished in 1.07ms
| src/XRandoms.sol:randomPool contract |                 |      |        |      |         |
|--------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                      | Deployment Size |      |        |      |         |
| 899135                               | 4523            |      |        |      |         |
| Function Name                        | min             | avg  | median | max  | # calls |
| randomNumber                         | 558             | 558  | 558    | 558  | 1       |
| randomWord                           | 8912            | 8912 | 8912   | 8912 | 1       |
| returnIndex                          | 8293            | 8442 | 8442   | 8591 | 2       |



Ran 1 test suites: 4 tests passed, 0 failed, 0 skipped (4 total tests)
```

Here is gas report after patched.

```
$ forge test --contracts test/randomPool.t.sol --gas-report
...

Running 4 tests for test/randomPool.t.sol:RandomPoolTest
[PASS] testFailReturnIndexOutOfBounds() (gas: 5418)
[PASS] testRandomNumber() (gas: 5787)
[PASS] testRandomWord() (gas: 9461)
[PASS] testReturnIndex() (gas: 9811)
Test result: ok. 4 passed; 0 failed; 0 skipped; finished in 1.22ms
| src/XRandoms.sol:randomPool contract |                 |      |        |      |         |
|--------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                      | Deployment Size |      |        |      |         |
| 2497178                              | 4441            |      |        |      |         |
| Function Name                        | min             | avg  | median | max  | # calls |
| randomNumber                         | 558             | 558  | 558    | 558  | 1       |
| randomWord                           | 3736            | 3736 | 3736   | 3736 | 1       |
| returnIndex                          | 382             | 1891 | 1891   | 3401 | 2       |



Ran 1 test suites: 4 tests passed, 0 failed, 0 skipped (4 total tests)
```
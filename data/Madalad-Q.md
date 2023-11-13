# QA Report

## `setCollectionCosts` and `setCollectionPhases` can be used to change key values during the minting phase

In `MinterContract`, function admins may call `setCollectionCosts` to set values used to determine the cost of minting at a particular timestamp, and `setCollectionPhases` to set the timestamps at which the allowlist/public minting phase begin and end.

These function must have been before users can call `mint`, however there is nothing stopping these setters from being called multiple times afterward during minting. Such functionality is not desirable as it could be used deceive and/or exploit users. Ensure that they are only callable once per collection, or that they cannot be called after `allowlistStartTime`.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157-L177

## `XRandoms#getWord` does not work properly

`XRandoms#getWord` chooses one of 100 words depending on the random `id` number passed as an argument, however the logic is incorrect. Since it is only called by the `randomWord` function, we know that `% 100` will have been applied to `id`, meaning it is between 0 and 99 inclusive, and never 100. This means that the word at index 0 ("Acai") has a 2/100 chance of being chosen, while the word at index 99 ("Watermelon") will never be chosen.

Change the function to always return `wordsList[id]` to ensure each word has an equal chance of being chosen.

```diff
File: smart-contracts\XRandoms.sol

    function getWord(uint256 id) private pure returns (string memory) {
        
        // array storing the words list
        string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", 
        "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine", 
        "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian", 
        "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum", 
        "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango", 
        "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple", 
        "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma", 
        "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];

        // returns a word based on index
-       if (id==0) {
-           return wordsList[id];
-       } else {
-           return wordsList[id - 1];
-       }

+       return wordsList[id];
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28-L32

## Insecure source of randomness used

Achieving randomness by hashing block constants, as is the method used by `RandomizerNXT`, is insecure because the values can be predicted. Moreover, miners/validators who control the block have the ability to manipulate certain values which can be abused to make sure the random value is in their favour.

The Additional Context section of the contest README states that `RandomizerNXT` will be used over `RandomizerRNG` and `RandomizerVRF` when the project is deployed. Reconsider this choice if true verifiable randomness is desired.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L57
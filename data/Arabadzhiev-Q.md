# [L-01] Missing or equal comparisons in one of the if statements in the `MinterContract::getPrice` function will lead to bumping up of the price to the maximum at the very end of the sale

## Lines Of Code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L540

## Impact
Users will be disincentivised from participating in the NFT collection sale at the end of it.

## Vulnerability Detail
Instead of returning a price that is equal to the end (lowest) price for the given NFT collection sale, the `getPrice` function will return the start (highest) price, in turn, disincentivising users from buying NFTs at the very end of the sale. This is due to the **less than** comparison not being **less than or equal to**. The **greater than** should not lead to any bad side effects, but it should still be changed to **greater than or equal to** for the sake of consistency.

## Recommended Mitigation Steps
Remove the **greater than** and **less than** operators and replace them with a **greater than or equal to** and **less than or equal to** operators.

```diff
function getPrice(uint256 _collectionId) public view returns (uint256) {
        uint tDiff;
        if (collectionPhases[_collectionId].salesOption == 3) {
            // increase minting price by mintcost / collectionPhases[_collectionId].rate every mint (1mint/period)
            // to get the price rate needs to be set
            if (collectionPhases[_collectionId].rate > 0) {
                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
            } else {
                return collectionPhases[_collectionId].collectionMintCost;
            }
-       } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
+       } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp >= collectionPhases[_collectionId].allowlistStartTime && block.timestamp <= collectionPhases[_collectionId].publicEndTime){
```

# [L-02] The chance of returning the first random word from the `randomPool::randomWord` function is double compared to the others and the chance of returning the last word is 0

## Lines Of Code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/XRandoms.sol#L28-L32

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/XRandoms.sol#L41

## Impact
The chance of returning the first word is double the chance of returning any other word, while the chance of returning the last word is none, which theoretically makes the function "non-random".

## Vulnerability Detail
In the `randomPool::randomWord` we are encoding a bunch of values, then computing their `keccak256` hash, then casting that to `uint256` and then finally modulo dividing that value by 100. This means that in the end of all of that, we are going to get a number between **0 and 99**. After we get that number, we pass it on to the `randomPool::getWord` function. However in that function, we see the following piece of code: 

```solidity
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1]; // @QA 0 has double chance of being selected while 99 will never be selected
        }
```

As we can see, there is special logic that handles the cases where the passed in `id` equals 0 and where it does not equal 0. Since in that logic, when the `id` value equals 0, we take it as it is, while in the other case we subtract 1 from it, the chance of returning the first word from `wordsList` is 2x, while the chance of returning the last word is none ,since the max value of `id` is 99, while the last index ot the `wordsList` array is also 99.

## Recommended Mitigation Steps
Remove the special zero handling logic in `randomPool::getWord` and treat all values of `id` the same way.

```diff
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
# [N-01] The `NextGenRandomizerVRF` inherits `Ownable`, but it does not utilize any of its functionality

## Lines Of Code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerVRF.sol#L19

## Recommendation
Since none of the functionality from the `Ownable` contract is being used, it would be better to just remove it, in order to both reduce the complexity of the contract and save some gas on deployment.

# [N-02] Unused `Ownable` import

The following `Ownable.sol` import is not being used anywhere within the `RandomizerNXT` contract and would be better of being removed.

## Lines Of Code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L15




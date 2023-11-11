# QA

1. [QA](#qa)
   1. [Low mostly lack validation check](#low-mostly-lack-validation-check)
      1. [1. `setCollectionCosts()` lack require check `require(rate < mintCost - endMintCost)` to prevent revert during public sale](#1-setcollectioncosts-lack-require-check-requirerate--mintcost---endmintcost-to-prevent-revert-during-public-sale)
      2. [2. `setCollectionCosts()` does not check `collectionEndMintCost < collectionMintCost` to prevent revert](#2-setcollectioncosts-does-not-check-collectionendmintcost--collectionmintcost-to-prevent-revert)
      3. [4. admin function `MinterContract.mintAndAuction()` should also check if `collectionPhase` information already initialize first](#4-admin-function-mintercontractmintandauction-should-also-check-if-collectionphase-information-already-initialize-first)
      4. [5. `getPrice()` sale option 3 (price increase for each sale) does not work when `rate > mintCost`](#5-getprice-sale-option-3-price-increase-for-each-sale-does-not-work-when-rate--mintcost)
      5. [6. `XRandoms.randomWord()`  return same word on same block](#6-xrandomsrandomword--return-same-word-on-same-block)

## Low mostly lack validation check to prevent revert later

### 1. `setCollectionCosts()` lack require check `require(rate < mintCost - endMintCost)` to prevent revert during public sale

It can revert [here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553) for sale option 2 (price drop linearly to zero). When `rate > mintCost - endMintCost`, it just mean price drop to zero in single `timePeriod`. Because time period can be set to anytime, and rate can be higher than sale cost. `getPrice()` should also support this case `rate > mintCost - endMintCost`.

### 2. `setCollectionCosts()` does not check `collectionEndMintCost < collectionMintCost` to prevent revert

### 4. admin function `MinterContract.mintAndAuction()` should also check if `collectionPhase` information already initialize first

before airdrop to [prevent underflow revert here](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L287). `mintAndAuction()` depend on using `collectionPhase` variable that might not be initialize yet. Include `require(setMintingCosts[_collectionID] == true, "Set Minting Costs");` check like in `mint()` function should prevent confusion. Because function revert when using zero value so no harm done

### 5. `getPrice()` sale option 3 (price increase for each sale) does not work when `rate > mintCost`

`setCollectionCosts()` should include require check to prevent this or price will forever stuck in starting price.

When rate > mintCost, it will cause division to return zero. so price multiplier always return 0 for all case.
<https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536>

### 6. `XRandoms.randomWord()`  return same word on same block

<https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L41>
`block.prevrandao` does not change on same epoch. So on same block, `XRandoms.randomWord()` will return same word.
This does not affect implementation `RandomizerNXT.sol` when calculate random hash for token.

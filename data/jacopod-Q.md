# Miss-configured collection costs can make all mints revert

## Description
The function `MinterContract::setCollectionCosts()` lacks input validation for the input argument `_timePeriod`, which sets `collectionPhases[_collectionID].timePeriod`. This parameter is used in multiple places as a divisor, so if the input argument is 0, it will cause division by zero errors in multiple minting functions:
+ [`mint()`](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L249)
+ [`mintAndAuction()`](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L292)
+ [`getPrice()`](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L546), which is used pretty much by every minting function

Notice that the `timePeriod` parameter is only used for certain values of `salesOption` (i.e., for certain sales modes). This makes it prone to set it to 0 when it is not required and makes it more likely to set it to 0 even when actually needed.

## Mitigation
Include the following requirement in the function `MinterContract::setCollectionCosts()`:

    require(_timePeriod > 0, "Invalid _timePeriod");
# Length in loop (gas optimisation)
## Rating
None. Gas optimisation.
## Smart contract 
AuctionDemo.sol, MinterContract.sol, and NextGenAdmin.sol
## Observation
The functions 
* “returnHighestBid”,  “returnHighestBidder”, and “cancelAllBids” in AuctionDemo 
* “airDropTokens” in MinterContract, and
* “registerBatchFunctionAdmin” in NextGenAdmin
will consistently access and load the length parameters for each loop iteration.
## Recommended mitigation steps
In cases where the number of loop iterations does not need to be dynamically determined, it is advisable to calculate the maximum number of iterations before the loop begins in order to save gas.

# Not necessary variable calculations / checks (gas optimisation)
## Rating
None. Gas optimisation.
## Observation
The following calculations and checks are redundant:
* https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L223, since covered by the following line L224
* https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267, since same as variable “collectionTokenMintIndex” in line L264
* https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L281, since same as variable  “collectionTokenMintIndex” in line L279
## Recommended mitigation steps
To save on gas costs, it's worth considering the removal of redundant calculations and checks.

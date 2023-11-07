Function returnHighestBid() is called whenever paricipateToAuction() is called. 

But returnHightestBid has `for loop` through previous bids. it causes gas cost getting increased according to more bidders.

## Recommendation
Set `highestBid` variable on the contract. and update it only whenever calling paricipateToAuction() or cancelBid().
By this means, we don't need to use `for loop` through all bids. and it will save gas.

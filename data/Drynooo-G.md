[G-1]
In the auctionDemo contract
In the returnHighestBid and returnHighestBidder functions, it is possible to save a lot of gas by not using loops. because when bidding, it will always be needed to be higher than the current highest bid, which will then be pushed at the end of the array. Therefore, the last bit of the array is the highest price, so no looping is needed.

[G-2]
When a new highest bid is generated, it is possible to refund the funds of the previous bids, as they no longer have a meaningful presence in the contract. This reduces the consumption of gas when users cancelAllBids.
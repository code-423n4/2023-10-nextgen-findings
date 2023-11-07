### Issue on https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L117

Param to Event Refund argument is not correct.
`emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);`
Here is passing 'highestBid' so it will be notified like the bidder have been refunded by more than his bid amount. 

I think, Correct implementation is like below
`emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, auctionInfoData[_tokenid][i].bid);`
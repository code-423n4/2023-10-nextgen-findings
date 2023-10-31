## Detail

The auctionInfoData list using in auctionDemo.sol always sorted. Because of check if it's bigger than the highest bid in participateToAuction().

```
...
    function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }

```

returnHighestBidder() which is called from claimAuction(), iterating auctionInfoData to find valid highest bidder. 

The Highest bidder will be latest bidder who have status is true. This mean `auctionInfoData[_tokenid][i].bid > highBid` is meaningless code and even it dosen't be updated during iterating - It's always 0. 

```
...

    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
...
```

## Recommended Mitigation Steps

If participateToAuction() and design of auctionInfoData won't be changed, Then I recommend

1. get auctionInfoData[_tokenid].length
2. iterate from the end (length-1)
3. check only auctionInfoData[_tokenid][i].status == true

and delete `uint256 highBid = 0;` and `auctionInfoData[_tokenid][i].bid > highBid`. 

This recommend will imporve your code and make gas saving
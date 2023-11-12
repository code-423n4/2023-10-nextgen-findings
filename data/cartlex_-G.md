
# [G-01] Function `returnHighestBid()` can be more optimized.

```diff
function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
    uint256 index;
    if (auctionInfoData[_tokenid].length > 0) {
        uint256 highBid = 0;
++      auctionInfoStru[] memory auctionInfo = auctionInfoData[_tokenid];
--      for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
++      for (uint256 i=0; i< auctionInfo.length; i++) {
--        if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
--              highBid = auctionInfoData[_tokenid][i].bid;
++          if (auctionInfo[i].bid > highBid && auctionInfo[i].status == true) {
++              highBid = auctionInfo[i].bid;
                index = i;
            }
        }
--          if (auctionInfoData[_tokenid][index].status == true) {
++          if (auctionInfo[index].status == true) {
            return highBid;
        } else {
            return 0;
        }
    } else {
        return 0;
    }
}
```

Gas benchmarks of `returnHighestBid()` function.

|          | avg      | median   | max      |
|----------|----------|----------|----------|
| Before   | 3720     | 3371     | 5584     |
| After    | 2696     | 2696     | 2696     |


# [G-02] Function `returnHighestBidder()` can be more optimized.

```diff
function returnHighestBidder(uint256 _tokenid) public view returns (address) {
    uint256 highBid = 0;
    uint256 index;
++  auctionInfoStru[] memory auctionInfo = auctionInfoData[_tokenid];
++  for (uint256 i=0; i< auctionInfo.length; i++) {
++      if (auctionInfo[i].bid > highBid && auctionInfo[i].status == true) {
--  for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
--      if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
            index = i;
        }
    }
++  if (auctionInfo[index].status == true) {
++      return auctionInfo[index].bidder;
--  if (auctionInfoData[_tokenid][index].status == true) {
--      return auctionInfoData[_tokenid][index].bidder;
    } else {
        revert("No Active Bidder");
    }
}
```
Gas benchmarks of `returnHighestBidder()` function.

|          | avg      | median   | max      |
|----------|----------|----------|----------|
| Before   | 2475     | 2475     | 2475     |
| After    | 1754     | 1754     | 1754     |
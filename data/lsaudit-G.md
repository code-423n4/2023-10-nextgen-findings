# [G-01] `AuctionDemo` can be fully redesigned

There are multiple of ways in which `AuctionDemo.sol` should be optimized. In this report, we have divide optimizations into sub-sections.

## Do not calculate highest bidder and highest bid every time during `participateToAuction()`.

In function `participateToAuction()` - every time the new bid is made - there's a call to `returnHighestBid(_tokenid)`.
This function iterates over the whole `auctionInfoData` mapping to find the value of the highest bid. This is extremely ineffective.


Much better idea, would be to declare new, public variable: `uint public highestBidder` and `uint public highestBid` and store the highest bidder and highest bid there.
That way, we won't need to iterate over `auctionInfoData` on every `participateToAuction()` call


```
uint public highestBidder;
uint public highestBid;

 function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > highestBid && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
        highestBidder = msg.sender;
        highestBid = msg.value;
    }
```

## Re-check who's the highest bidder only when `cancelBid()` or `cancelAllBids()` is made.

Please notice, that `participateToAuction()` will always set highest bidder to a new one. 
This check: `require(msg.value > highestBid)`, basically implies that when someone calls `participateToAuction()` - then he/she needs to outbid current highest bidder, so the caller will become new highest bidder.

The only update for the highest bidder/highest bid is needed, when we call `cancelBid()` or `cancelAllBids()`.
Let's check the example to understand how it really work:

```
0xAAA calls participateToAuction() with 1000
0xAAA is now the highest bidder with the highest bid: 1000

0xBBB calls participateToAuction() with 2000
0xBBB becomes the highest bidder with the highest bid: 2000

0xCCC calls participateToAuction() with 1500
1500 < 2000, thus participateToAuction() reverts and the highest bidder is still 0xBBB

0xDDD calls participateToAuction() with 3000
0xDDD becomes the highest bidder with the highest bid: 3000

0xDDD calls cancelBid()
Now we need to update the highest bidder (because 0xDDD is not highest bidder anymore) and update the highest bid (because 3000 is not highest bid anymore)

```

This implies, that the re-calculation of highest bidder and highest bid should be done only in `cancelBid()` and `cancelAllBids()`:


```
function cancelBid(uint256 _tokenid, uint256 index) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
        auctionInfoData[_tokenid][index].status = false;
        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
        highestBidder = returnHighestBidder(_tokenid);
        highestBid = returnHighestBid(_tokenid);
    }
```

```
function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }

        highestBidder = returnHighestBidder(_tokenid);
        highestBid = returnHighestBid(_tokenid);
    }
```

After making those changes, we can also update `claimAuction()`, since we don't need to calculate highestBidder there anymore:

```
function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = highestBid;
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = highestBidder;
        (...)
```

## Optimize how highest bidder and higest bid is being calculated

Currently, both `returnHighestBidder()` and `returnHighestBid()` iterates over `auctionInfoData` and check for the highest bidder.
Please notice, that whenever we add the new highest bidder in `participateToAuction()`, it's being added to the end of the `auctionInfoData[_tokenid]`:

```
60:		auctionInfoData[_tokenid].push(newBid);
```

This implies, that the highest bidder/highest bid will be at the end of the array.
It would be much better idea to iterate `auctionInfoData[_tokenid]` not from 0 to `auctionInfoData[_tokenid].length`, but from `auctionInfoData[_tokenid].length` to 0.
We will need to iterate until we find a bidder with `status` set to `true`.
Let's visualize this example:

```
0xA bids with 100
0xB bids with 200
0xC bids with 300
0xD bids with 400
0xE bids with 500
```
`auctionInfoData[_tokenid] = [100, 200, 300, 400, 500]`
Whenever someone calls `participateToAuction()` and doesn't outbid, function will revert and that bid won't be added to the array. According to this, we can be sure, that the last element of the array is indeed the highest bidder.
This may only change when users will start to cancel their bids:

```
0xD cancels
0xB cancels
0xE cancels
```

`auctionInfoData[_tokenid]` still contains  `[100, 200, 300, 400, 500]` but some of those bids have `status` set to false (because of the `cancelBid()`):

```
[100:(status: true), 200:(status: false), 300:(status: true), 400:(status: false), 500:(status: false)]
```

When we remove all the status set to false, the highest bidder will be still in the last element of the array:
```
[100, 300]
```

However, removing every bid with `status` false won't be very effective. Instead, we should iterate from the end of the `auctionInfoData[_tokenid]`, until we find the element with `status` set to `true`. It will be the highest bid and the highest bidder.

## Merge `returnHighestBidder()` and `returnHighestBid()` into one function

Both functions iterates over the same array and returns data at the same index. This means, that these functions can be merged in to a single function `returnHighestBidderAndBid()`:

```
(highestBidder, highestBid) = returnHighestBidderAndBid()
```

## Do not iterate over whole `auctionInfoData` in `claimAuction()`

When auction is finished, calling `claimAuction()` allows to refund every non-winning bid. We're basically iterate over the whole `auctionInfoData[_tokenid]`.
When most of the users had already refunded their bid (by calling `cancelBid()`/`cancelAllBids()` before auction finishes) - this is extremely ineffective. We need to re-implement the whole mechanism.

It will be much more effective to implement two new functions: `auctionClaimAndGetNFT()` which will be responsible only for claiming NFT by the auction winner; `refundBids()`, which will basically be the same as `cancelAllBids()` but it will allow to refund after auction. Instead of iterate over the whole array - each user will call this function for their addresses.


# [G-02] `MinterContract.sol` - do not calculate `viewCirSupply()` during every loop iteration

[File: MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L234-L237)
```
for(uint256 i = 0; i < _numberOfTokens; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
        }
```

can be rewritten to:

```
uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
for(uint256 i = 0; i < _numberOfTokens; i++) {
 gencore.mint(mintIndex + i, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
        }
```


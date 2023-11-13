# Lines of code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L134-L143
# Vulnerability details 
## Impact 
The ```cancelAllBids``` function is public and can be called by anyone. The lack of access control allows bids to be easily canceled by anyone.But the impact is very low because only msg.sender can cancel his bid and get his money. It won't cancel other bids.
## Proof of Concept 
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
    }
``` 
## Tools Used 
Manual 
## Recommended Mitigation Steps 
Recommend using proper access control, for example: only collection admin or only protocol admin. The function should be rewrited.
```
function cancelAllBids(uint256 _tokenid) public AdminRequired {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            }
        }
    }
```
## Assessed type 
Access Control
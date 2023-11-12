
# [L-01] Validating ``tokenid``

The function ParticipateToAuction accepts a bid without checking if the `tokenid` corresponds to a valid token. This could allow users to bid on a null tokenid "0" that does not belong to any token. The users would lose gas fees by executing a futile transaction. The function should validate the `tokenid` before the require statement to prevent this problem.

```
function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L57


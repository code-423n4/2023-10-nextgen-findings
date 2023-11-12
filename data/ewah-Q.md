
# [L-01] Validating ``tokenid``

The function ``participateToAuction`` accepts a bid without checking if the ``tokenid`` corresponds to a valid token. This could allow users to bid on a null ``tokenid`` “0” that does not belong to any token. The users would lose gas fees by executing a futile transaction. To prevent this problem, the function should validate the ``tokenid`` before the require statement. One way to do this is to check if the ``tokenid`` is less than or equal to the total circulation of the tokens

```
function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L57


## [L-02] Function ``participateToAuction`` Does Not Emit Any Events When a New Bid is Placed Or When An Auction is Completed

The function ``participateToAuction`` does not emit any events when a new bid is placed or when an auction is completed. Events are useful for notifying the users and other contracts about the changes in the state of the contract, and for logging the history of the transactions.

```
function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L57

```
// Define the event types
event NewBid(uint256 indexed tokenid, address indexed bidder, uint256 amount);
event AuctionEnded(uint256 indexed tokenid, address indexed winner, uint256 amount);
```


# [L-01] Validating ``tokenid``

The function ``participateToAuction`` accepts a bid without checking if the ``tokenid`` corresponds to a valid token. This could allow users to bid on a null ``tokenid`` “0” that does not belong to any token. The users would lose gas fees by executing a futile transaction. To prevent this problem, the function should validate the ``tokenid`` before the require statement and checking if the ``tokenid`` is greater than "0" and also less than or equal to the total circulation of the tokens

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

### [L-03] Validating ``tokenid``

The function ``returnHighestBid`` lacks validation of tokenid, which could lead to incorrect or misleading results. The function returns the highest bid amount for a given tokenid that is being auctioned. However, if the ``tokenid`` is not valid, the function will return zero as the highest bid amount. This is because the function will check if the array ``auctionInfoData[_tokenid]`` has any elements before proceeding with the loop. If the array is empty or undefined, the function will return zero without entering the loop. If the tokenid is zero, the function will also return zero, because the zero id is null ``tokenid``. Moreover, the users who call the function with an invalid tokenid will lose gas fees by executing a futile transaction. To prevent this problem, the function should validate the ``tokenid`` before accessing the array ``auctionInfoData[_tokenid]``. One way to do this is to check if the ``tokenid`` is greater than "0" and also less than or equal to the total circulation of the tokens.

```
function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L65.


#### [L-03] Function ``returnHighestBid`` Doesn't Return The highestBid Index. 

Function ``returnHighestBid``. does not return the index of the highest bid, only the amount. This means that the caller of the function cannot know who made the highest bid, which is important for the auction logic to return the index of the user because it can help identify the bidder who made the highest bid. The index can be used to access the bidder’s address and other information from the ``auctionInfoData``. 

```
function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87


#####[L-04] Check ``returnHighestBidder`` Array Before Looping. 

Function ``returnHighestBidder`` does not check if the ``auctionInfoData[_tokenid]`` array is greater than zero before looping through it. This could result in an error if the array is empty, as the index variable would be undefined and the function would try to access an invalid element of the array.Adding a condition or a require statement to check if the array is not empty before looping through will create a positive impact on the function. Preventing the function from throwing an error or reverting the transaction if the array is empty, which could cause inconvenience and frustration for the users. It will also improve the security and efficiency of the function, as it will not waste gas or resources on an invalid operation. 

```
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
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87
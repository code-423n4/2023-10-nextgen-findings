
# L-01 :  Summing up all the bids and sending the fund in one transaction is a better way to transfer funds 
The `cancelAllBids` function is for cancelling all the bids and getting the refund. But here for every bid a low level call is executed to transfer eth ,  which is not efficient design . Its a better design choice to sum up all the bids and sending them in one txn .  It'll also save a lot of gas . 
```solidity 
  function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false; //@ci low  better to sum up all the bids and pay them at the end 
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L134

## recommendation 
Rewrite the ` cancelAllBids` function in  this way : 
```diff
  function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
+       uint256 totalAmount;
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false; 
-               (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
+               totalAmount = totalAmount + auctionInfoData[_tokenid][i].bid ;
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }
+       (bool success, ) = payable(msg.sender).call{value: totalAmount}("");
}
```
# L-02: No event emmission for participating in auction ! 
`participateToAuction ` function is a crucial function in the auction contract . However , this functioni doesnot emit any event for placing a bid . 
```solidity 
  function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
//@audit NO event ??? 
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57C3-L61C6

## recommendation : 
declare this event : 
```solidity 
event PlacedBid((address indexed _add, uint256 indexed tokenid, uint256 indexed funds); 
```
And emit this event when any bidder placed a bid : 
```diff 

  function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
+       emit PlacedBid(msg.sender ,_tokenid ,msg.value );  
}
    
```

# N-01 : Constructor cant be public ! 
Visibility for constructor should be  ignored  .
```solidity 
 constructor (address _minter, address _gencore, address _adminsContract) public { //@ci  public  ? 
        minter = IMinterContract(_minter);
        gencore = _gencore;
        adminsContract = INextGenAdmins(_adminsContract);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L36
## recommendation 
Remove `public` visibility  from the constructor . 

# N-02 : The `highBid ` variable should be named `highestBid ` as it represents highestBid . 
The `highBid ` variable should be named `highestBid ` as it represents highestBid . 
```solidity 
 if (auctionInfoData[_tokenid][index].status == true) { 
                return highBid;// @ci low  should be named highestBid ! 
            }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L76


# N-03 :  Codebase lacks documentation  
Documentaion is crucial for auditors and developers to get the code right in future purpose . But the whole codebase lacks inline comments and detailed documentation which makes it tough for and auditor to understand the code .  Also  proper documentation is necessary for maintaining a codebase 




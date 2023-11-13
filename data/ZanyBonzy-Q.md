### **[AuctionDemo.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol)**
   
 1. No need to denote the constructor's visibility. [L36](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L36)

   `constructor (address _minter, address _gencore, address _adminsContract) public {`

2. Consider removing empty/unused `else` block. [L118](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L118), [L141](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L141)

  `else {}`
  
3. `returnhighestbidder` sets the highbid to 0. The available bids are then compared against it, presumably to check if they're higher. However, the highbid is not set to the last checked bid. This means that that bids are compared against 0 and not the actual bids. COnsider fixing this for as good practice.
```
      function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) { //@note all bids are compared against 0
                index = i; 
            }
        }
        ...
   ```

4. `returnHighestBidder` function can be refactored. 
 To participate in an auction, the amount bid must be greater than the current highest bid, and each `newbid` is pushed into the `auctionInfoData` array.  This ensures that every `newbid` (which is greater than the previous) becomes the last element of the `auctionInfoData` array. Thererfore, the last element of the array belonds to the highest bidder.

The `returnHighestBidder` can be made to count down from the last element instead of the first, and checks their auction status. If true, then it is selected. It seems faster and less gas intensive as the loop is more likely to run lesser iterations.
e.g
```
function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 index;
        uint256 refactor = auctionInfoData[_tokenid].length;
for (uint i = refactor; i > 0;) {
    if (auctionInfoData[_tokenid][i - 1].status == true) { 
        index = i - 1;
        break;
    unchecked{
    		--i;
    	}
    else {
   revert("No Active Bidder"); 
   }
}
return auctionInfoData[_tokenid][index].bidder;
}

```
5. Potential race condition
   The `participateToAuction`, `cancelBid`, `cancelAllBids` functions can be called when `block.timestamp <= minter.getAuctionEndTime(_tokenid)` i.e when auction ends.

  The `claimAuction` function can be claimed when `block.timestamp >= minter.getAuctionEndTime(_tokenid)`.

  This introduces a potenctial race condition in which users and admins can all call the functions at the exact same time for the exact same token, when `block.timestamp >= minter.getAuctionEndTime(_tokenid)`.

  Depending on which gets executed first, undersired outcomes may occur. For instance, If the highest bidder decides to cancel his auction, he might find out that the admin already claimed on his behalf, or that another user outbidded him at the last second. 
  Consider changing the condition in `claimAuction` to `block.timestamp > minter.getAuctionEndTime(_tokenid)`.

### [NextGenAdmns.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L58C5-L61C6)
 
6.   [`registerCollectionAdmin`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L58C5-L61C6) should check for collection existence before registering admin so as to prevent setting admin for non-existent collections;
  ```
function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
        require(_collectionID > 0, "Collection Id must be larger than 0"); //@note inexistent collection??
        collectionAdmin[_address][_collectionID] = _status;
    }
 ```
7.  Admins can be registered, but not deregistered. Cosider introducting functions to deregister admins in case they get compromised or go rouge.
`   function registerAdmin(address _admin, bool _status) public onlyOwner {`

`   function registerFunctionAdmin(address _address, bytes4 _selector, bool _status) public AdminRequired {`

`    function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {`
     
`    function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {`

### [XRandoms.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol)
8. `abi.encodePacked` should not be used with dynamic types when passing the result to a hash function such as `keccak256`. Use `abi.encode` instead, which will pad items to 32 bytes, to prevent any hash collisions.

`uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;` [L36](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L36)

`uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;` [L41](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L41)

### [MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol)
9. External call in unbounded loop
   `uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);`[L188](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L188)

10. Typo - "greater" not "grater"

    `require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");`

12. Misleading revert string - "at least 1 mint/period" better

    `require(tDiff>=1, "1 mint/period");`
    
### RandomizerNXT.sol
13. updateAdminsContract doens't check if admin address is registered
```
   function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {
        adminsContract = INextGenAdmins(_admin);
    }
```

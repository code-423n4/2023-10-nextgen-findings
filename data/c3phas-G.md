# Codebase Optimization Report

## Table of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Winner of the bid shouldn't have to loop on all other bidders in order to get his winnings](#winner-of-the-bid-shouldnt-have-to-loop-on-all-other-bidders-in-order-to-get-his-winnings)
  - [Pack structs by reducing the size of this variables(Save 1 SLOT: 2100 Gas) - not found by bot](#pack-structs-by-reducing-the-size-of-this-variablessave-1-slot-2100-gas---not-found-by-bot)
  - [We can pack this struct by reducing the size  of variables to `uint128`(Save 1 SLOT: 2100 Gas) - not found by bot](#we-can-pack-this-struct-by-reducing-the-size--of-variables-to-uint128save-1-slot-2100-gas---not-found-by-bot)
  - [Pack struct by reducing the size for timestamps(Save 1 SLOT: 2100 Gas) - not found by bot](#pack-struct-by-reducing-the-size-for-timestampssave-1-slot-2100-gas---not-found-by-bot)
  - [Redundant SLOAD inside the constructor(Save 2100 Gas)](#redundant-sload-inside-the-constructorsave-2100-gas)
  - [Avoid zero to one storage writes](#avoid-zero-to-one-storage-writes)
  - [Caching done in the wrong way](#caching-done-in-the-wrong-way)
  - [We can save 2 SSTORES in some cases(4200 Gas)](#we-can-save-2-sstores-in-some-cases4200-gas)
  - [Avoid an SLOAD here(save ~100 Gas)](#avoid-an-sload-heresave-100-gas)
  - [We can save 1 SLOAD by inlining the modifier](#we-can-save-1-sload-by-inlining-the-modifier)
  - [Conclusion](#conclusion)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.


## Winner of the bid shouldn't have to loop on all other bidders in order to get his winnings

Currently, the only way to claim your winnings is by looping on all those who participated until we find the winning index. This is too expensive for the winner of the bids. Since we already know the winning bidder from `returnHighestBidder(_tokenid);` we should just send it 

```diff
+    modifier onlyWinner(uint256 _tokenId){
+        require(msg.sender == returnHighestBidder(_tokenId));
+        _;
+    }

     constructor (address _minter, address _gencore, address _adminsContract) public {
         minter = IMinterContract(_minter);
@@ -97,7 +101,20 @@ contract auctionDemo is Ownable {
             } else {
                 revert("No Active Bidder");
         }
-    }

+
+  //the modifier ensures the msg.sender is in fact the highest bidder
+    function claimAuction(uint256 _tokenid) public onlyWinner(_tokenid){
+        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
+        auctionClaim[_tokenid] = true;
+        uint256 highestBid = returnHighestBid(_tokenid);
+        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
+        IERC721(gencore).safeTransferFrom(ownerOfToken, msg.sender, _tokenid);//@audit-info send the item to winenr
+        (bool success, ) = payable(owner()).call{value: highestBid}("");//@audit-info send the amount to owner of item
+        emit ClaimAuction(owner(), _tokenid, success, highestBid);
+
+
+    }
```


The original `claimAuction` function should be renamed to `refundBids()` and be callable by only `Admin` . It should only handle the case of those who did not win, so we can remove the logic for the winning bid from there

```diff
-    modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
-      require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
+    modifier AdminRequired(uint256 _tokenId, bytes4 _selector) {
+      require( adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
       _;
     }


-    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
+    function refundBid(uint256 _tokenid) public AdminRequired(_tokenid,this.claimAuction.selector){
         require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
         auctionClaim[_tokenid] = true;
         uint256 highestBid = returnHighestBid(_tokenid);
         address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
         address highestBidder = returnHighestBidder(_tokenid);
         for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
-            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
-                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
-                (bool success, ) = payable(owner()).call{value: highestBid}("");
-                emit ClaimAuction(owner(), _tokenid, success, highestBid);
-            } else if (auctionInfoData[_tokenid][i].status == true) {
+            if (auctionInfoData[_tokenid][i].bidder != highestBidder && auctionInfoData[_tokenid][i].bid != highestBid && auctionInfoData[_tokenid][i].status == true) {//@audit-info you bid but didn't win,refund
                 (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
-                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
+                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, auctionInfoData[_tokenid][i].bid);
             } else {}
         }
-    }
```

**Additional Info: we could also allow the admin to initialize the claiming process if the winner doesn't call claim**
**We need to add some tests for the newly created functions**


## Pack structs by reducing the size of this variables(Save 1 SLOT: 2100 Gas) - not found by bot

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L63-L66
```solidity
File: /smart-contracts/MinterContract.sol
63:    struct royaltiesPrimarySplits {
64:        uint256 artistPercentage;
65:        uint256 teamPercentage;
66:    }
```
We can reduce the size of this variables to `uint128` as it would be big enough. When this variables are being set, we are constraining their values to be == 100, see https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L369-L376

```solidity
369:    function setPrimaryAndSecondarySplits(uint256 _collectionID, uint256 _artistPrSplit, uint256 _teamPrSplit, uint256 _artistSecSplit, uint256 _teamSecSplit) public FunctionAdminRequired(this.setPrimaryAndSecondarySplits.selector) {
370:        require(_artistPrSplit + _teamPrSplit == 100, "splits need to be 100%");
371:        require(_artistSecSplit + _teamSecSplit == 100, "splits need to be 100%");
372:        collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage = _artistPrSplit;
373:        collectionRoyaltiesPrimarySplits[_collectionID].teamPercentage = _teamPrSplit;
374:        collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage = _artistSecSplit;
375:        collectionRoyaltiesSecondarySplits[_collectionID].teamPercentage = _teamSecSplit;
376:    }
```
As you can see , we have some require statement that ensures the variables being set are always going to be 100

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..896229d 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -61,8 +61,8 @@ contract NextGenMinterContract is Ownable {
     // royalties primary splits structure

     struct royaltiesPrimarySplits {
-        uint256 artistPercentage;
-        uint256 teamPercentage;
+        uint128 artistPercentage;
+        uint128 teamPercentage;
     }
```

## We can pack this struct by reducing the size  of variables to `uint128`(Save 1 SLOT: 2100 Gas) - not found by bot

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L88-L91
```solidity
File: /smart-contracts/MinterContract.sol
88:    struct royaltiesSecondarySplits {
89:        uint256 artistPercentage;
90:        uint256 teamPercentage;
91:    }
```
The explanation is similar to the previous finding
```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..94bdfb0 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -86,8 +86,8 @@ contract NextGenMinterContract is Ownable {
     // royalties secondary splits structure

     struct royaltiesSecondarySplits {
-        uint256 artistPercentage;
-        uint256 teamPercentage;
+        uint128 artistPercentage;
+        uint128 teamPercentage;
     }

```


## Pack struct by reducing the size for timestamps(Save 1 SLOT: 2100 Gas) - not found by bot

**`setFinalSupplyTimeAfterMint`** is a timestamp and can be reduced to `uint40`
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L57
```solidity
File: /smart-contracts/NextGenCore.sol
44:    struct collectionAdditonalDataStructure {
45:        address collectionArtistAddress;
46:        uint256 maxCollectionPurchases;
47:        uint256 collectionCirculationSupply;
48:        uint256 collectionTotalSupply;
49:        uint256 reservedMinTokensIndex;
50:        uint256 reservedMaxTokensIndex;
51:        uint setFinalSupplyTimeAfterMint;
52:        address randomizerContract;
53:        IRandomizer randomizer;
54:    }

56:    // mapping of collectionAdditionalData struct
57:    mapping (uint256 => collectionAdditonalDataStructure) private collectionAdditionalData;
```


```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..b021132 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -48,7 +48,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
         uint256 collectionTotalSupply;
         uint256 reservedMinTokensIndex;
         uint256 reservedMaxTokensIndex;
-        uint setFinalSupplyTimeAfterMint;
+        uint40 setFinalSupplyTimeAfterMint;
         address randomizerContract;
         IRandomizer randomizer;
     }
```

## Redundant SLOAD inside the constructor(Save 2100 Gas)

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L108-L112
```solidity
File: /smart-contracts/NextGenCore.sol
108:    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:        adminsContract = INextGenAdmins(_adminsContract);
110:        newCollectionIndex = newCollectionIndex + 1;
111:        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:    }
```

On Line 110, we are making an SSTORE, where we write to the state variable `newCollectionIndex`. Since this is happening inside our constructor and the variable `newCollectionIndex` wasn't initialized during declaration, it means the variable has a value of `0` so no need to read it again

```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..355da91 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -107,7 +107,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // smart contract constructor
     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
         adminsContract = INextGenAdmins(_adminsContract);
-        newCollectionIndex = newCollectionIndex + 1;
+        newCollectionIndex = 1;
         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
     }
```

## Avoid zero to one storage writes

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

**Affected Instances**
We are setting the variable `newCollectionIndex` inside the constructor to be 1, as this variable is never decremented, only incremented, we can save some gas by initializing the variable during declaration as 1, which would help us avoid the 0 --> 1 initialization

```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..5d76ca3 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -23,7 +23,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     using Strings for uint256;

     // declare variables
-    uint256 public newCollectionIndex;
+    uint256 public newCollectionIndex = 1;

     // collectionInfo struct declaration
     struct collectionInfoStructure {
@@ -107,7 +107,6 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     // smart contract constructor
     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
         adminsContract = INextGenAdmins(_adminsContract);
-        newCollectionIndex = newCollectionIndex + 1;
         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
     }
```

## Caching done in the wrong way

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L415-L419
```solidity
File: /smart-contracts/MinterContract.sol
415:    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
416:        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
417:        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
418:        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
419:        uint256 royalties = collectionTotalAmount[_collectionID];
```
On Line 419 , we are caching the value of `collectionTotalAmount[_collectionID]` in the variable `royalties` yet we read the state variable earlier on Line 417. We can save 1 SLOAD by moving the cached line to the top

```diff
     function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
         require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
-        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
-        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
         uint256 royalties = collectionTotalAmount[_collectionID];
+        require(royalties > 0, "Collection Balance must be grater than 0");
+        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
         collectionTotalAmount[_collectionID] = 0;
         address tm1 = _team1;
         address tm2 = _team2;
```

## We can save 2 SSTORES in some cases(4200 Gas)

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L108-L112
```solidity
File: /smart-contracts/NextGenCore.sol
108:    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:        adminsContract = INextGenAdmins(_adminsContract);
110:        newCollectionIndex = newCollectionIndex + 1;
111:        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:    }
```

In this constructor, we are calling the function `_setDefaultRoyalty()` which has the following implementation
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/ERC2981.sol#L90-L101
```solidity
    function _setDefaultRoyalty(address receiver, uint96 feeNumerator) internal virtual {
        uint256 denominator = _feeDenominator();
        if (feeNumerator > denominator) {
            // Royalty fee will exceed the sale price
            revert ERC2981InvalidDefaultRoyalty(feeNumerator, denominator);
        }
        if (receiver == address(0)) {
            revert ERC2981InvalidDefaultRoyaltyReceiver(address(0));
        }


        _defaultRoyaltyInfo = RoyaltyInfo(receiver, feeNumerator);
    }
```
This function has some checks that must pass else we would revert. We can take advantage of this revert case to save quite a lot of gas.
Since in our constructor we first make some SSTORES(writing to storage: 2200 Gas per variable), we might loose all the gas used in SSTORE if one of the checks in the function `_setDefaultRoyalty()` is not met
There are two ways to refactor this code, either perform the function call earlier on in the constructor ie before doing any SSTORES or we could just inline the entire function. Let's work with the first one

```diff
@@ -106,9 +106,9 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {

     // smart contract constructor
     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
+        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
         adminsContract = INextGenAdmins(_adminsContract);
         newCollectionIndex = newCollectionIndex + 1;
-        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
     }
```

With this change, in case of a revert inside the function `_setDefaultRoyalty()` , we would not write to the two state variables.

## Avoid an SLOAD here(save ~100 Gas)

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L124-L130
```solidity
File: /smart-contracts/AuctionDemo.sol
124:    function cancelBid(uint256 _tokenid, uint256 index) public {
125:        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
126:        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
127:        auctionInfoData[_tokenid][index].status = false;
128:        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
129:        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
130:    }
```
On Line 126, we have a check `auctionInfoData[_tokenid][index].bidder == msg.sender`  and reverts if the check does not pass. If the check passes, it means `auctionInfoData[_tokenid][index].bidder` is equal to `msg.sender` and thus any occurrence of that state variable, we can replace it with `msg.sender`

```diff
diff --git a/smart-contracts/AuctionDemo.sol b/smart-contracts/AuctionDemo.sol
index 95533fb..cbcc140 100644
--- a/smart-contracts/AuctionDemo.sol
+++ b/smart-contracts/AuctionDemo.sol
@@ -125,7 +125,7 @@ contract auctionDemo is Ownable {
         require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
         require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
         auctionInfoData[_tokenid][index].status = false;
-        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
+        (bool success, ) = payable(msg.sender).call{value: auctionInfoData[_tokenid][index].bid}("");
         emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
     }
```

## We can save 1 SLOAD by inlining the modifier 

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L58-L61
```solidity
File: /smart-contracts/NextGenAdmins.sol
58:    function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
59:        require(_collectionID > 0, "Collection Id must be larger than 0");
60:        collectionAdmin[_address][_collectionID] = _status;
61:    }
```

The function `registerCollectionAdmin()` makes use of the modifier `AdminRequired` which has the following implementation
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L31-L34
```solidity
    modifier AdminRequired {
      require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
      _;
    }
```

This modifier reads the state variable `adminPermissions[msg.sender]` and checks if it's true.Inside our function `registerCollectionAdmin()` we have another check `_collectionID > 0` , we should be performing the cheap check first. To achieve this, we must inline the modifier as follows

```diff
diff --git a/smart-contracts/NextGenAdmins.sol b/smart-contracts/NextGenAdmins.sol
index dc68ddd..8f62ae1 100644
--- a/smart-contracts/NextGenAdmins.sol
+++ b/smart-contracts/NextGenAdmins.sol
@@ -55,8 +55,9 @@ contract NextGenAdmins is Ownable{

     // function to register a collection admin

-    function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
+    function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public  {
         require(_collectionID > 0, "Collection Id must be larger than 0");
+        require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
         collectionAdmin[_address][_collectionID] = _status;
     }
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.

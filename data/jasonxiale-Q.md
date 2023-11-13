# [L-01] `NextGenCore._mintProcessing` is violate Checks-Effects-Interactions rules
File:
    https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L227-L232
    https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L213-L223
    https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L178-L185
    https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L189-L200

`_mintProcessing` can call external within [_safeMint](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L231) function, and the functions `airDropTokens/mint/burnToMint` calling `_mintProcessing` doesn't have reentrancy protection, which makes those functions vulnerable to reentrancy attack 

# [L-02] `AuctionDemo.cancelAllBids` is inefficient.
File: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L134-L143
Function `AuctionDemo.cancelAllBids` is very inefficient, everytime a bidder calls this function to cancel his bids, the function will go through all the bids to find out the caller's bids.
To make the function much more efficient, we can setup a new mapping **mapping(uint256 => mapping(address => auctionInfoStru[]))** to record which tokenId a bidder has participated in, the first **uint** is used as index for collectionId, and the **address** stands for the bidder, and finally, the **auctionInfoStru[]** means the bids he offers

# [L-03] `AuctionDemo.returnBids` can be DOSed.
File:
    https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L147-L149
Function `AuctionDemo.returnBids` returns **auctionInfoStru[] memory**, so there are lots of bids, the return value can run out of gas.
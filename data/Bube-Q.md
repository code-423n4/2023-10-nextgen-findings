[1]

# Risk
Low

# Title
The input argument `_tokenid` is not checked if it is valid in `AuctionDemo.sol` contract

## Impact
In the functions: `participateToAuction()`, `returnHighestBid()`, `returnHighestBidder()`, `claimAuction()`, `cancelBid()`, `cancelAllBids()` and `returnBids()` in `AuctionDemo.sol` contract the input argument `_tokenid` is not checked if it is a valid token id.
Add a check in all functions in the contract `AuctionDemo.sol` that the `_tokenid` is valid.

## Links
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L57
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L65
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L104
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L124
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L134
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L147
## [L-1]： `retrievetokenImageAndAttributes`  don't check `_tokenId` exist or not.

https://github.com/code-423n4/2023-10-nextgen/blob/2467db02cc446374ab9154dde98f7e931d71584d/smart-contracts/NextGenCore.sol#L467-L469

https://github.com/code-423n4/2023-10-nextgen/blob/2467db02cc446374ab9154dde98f7e931d71584d/smart-contracts/NextGenCore.sol#L281

`retrievetokenImageAndAttributes` is to get token arrtributess,but it doesn't check  `_tokenId` exist or not.Event tokenid is minted,it also don't check `tokenImageAndAttributes` set or not. 
Acutually,in `mint` frocess, `updateImagesAndAttributes` is to set attributes never called. It's possible call `retrievetokenImageAndAttributes` get massive 0 value image data.


## [L-2] `mintAndAuction`  don't have `_auctionEndTime` check

https://github.com/code-423n4/2023-10-nextgen/blob/2467db02cc446374ab9154dde98f7e931d71584d/smart-contracts/MinterContract.sol#L276-L298

 `_auctionEndTime` never get checked in `mintAndAuction`  auction creation. When `_auctionEndTime`  < blocktime  or `_auctionEndTime` value too big, it will not enable the auction.


## [L-3] The excess tokens beyond what is required for minting should return to user.
https://github.com/code-423n4/2023-10-nextgen/blob/2467db02cc446374ab9154dde98f7e931d71584d/smart-contracts/MinterContract.sol#L233
When `mint`, user is possible to send more toknes than price, the excess tokens beyond what is required should return to user.

## [L-4]  typo error  `auctionInfoStru` >  `auctionInfoStruct`
https://github.com/code-423n4/2023-10-nextgen/blob/a6f2397b68ef2865374c1bf7629349f25e71a44d/smart-contracts/AuctionDemo.sol#L43

 typo error  `auctionInfoStru` >  `auctionInfoStruct`


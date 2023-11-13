# QA

## Shadowed variables(Not found by bot)

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L108-L112
```solidity
File: /smart-contracts/NextGenCore.sol
    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
        adminsContract = INextGenAdmins(_adminsContract);
        newCollectionIndex = newCollectionIndex + 1;
        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
    }
```

We are passing `name` and `symbol`  to  ERC721 which has the functions with similar names. We can rename the parameters here to `_name` and `_symbol`

## The code does not check the frozen state

According to the docs when we are calling `The _freezeCollection(..)`  `The collection should exist and not be frozen.`
However , in our implementation we only check that the collection exists

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292-L295
```solidity
File: /smart-contracts/NextGenCore.sol
292:    function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
293:        require(isCollectionCreated[_collectionID] == true, "No Col");
294:        collectionFreeze[_collectionID] = true;
295:    }
```

### Recommendation

Add a check for whether the collection is in frozen state

## We Should ensure the length of arrays is equal

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281-L288
```solidity
File: /smart-contracts/NextGenCore.sol
281:    function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
282:        for (uint256 x; x < _tokenId.length; x++) {
283:            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
284:            _requireMinted(_tokenId[x]);
285:            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
286:            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
287:        }
288:    }
```

We should ensure `_tokenId.length` == `_images.length` == `_attributes.length_`


## No need to specify visibility for constructor

Visibility for constructors are not necessary/ they are usually ignored
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L36-L40
```solidity
File: /smart-contracts/AuctionDemo.sol
    constructor (address _minter, address _gencore, address _adminsContract) public {
        minter = IMinterContract(_minter);
        gencore = _gencore;
        adminsContract = INextGenAdmins(_adminsContract);
    }
```

### Recommendation

We can simply get rid of the visibility specify

## The function name does not depict the true functionality being implemented

The function is named `claimAuction` but during the claiming process, we are also making refunds to bidders who didn't win the auction.
The function name only depicts the claiming functionality and misses the refunds.


https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L104-L120
```solidity
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = returnHighestBidder(_tokenid);
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }
```

### Recommendation

To clearly depict what is happening, we suggest splitting the function into two, named `claimAuction()`  which handles the claiming part(for the winner of the bid) and the other function `refunds()`  which will take care of refunding the other bidders


## No way for winning bidder to change their address if the current one is compromised

When claiming the Auction, there are two ways of finalizing it, one a winner claims his winnings and two, we can have the admin initializing the claiming in which case we loop and send the winning bidder his reward while also refunding the bidders who didn't win

In a scenario where the winner of the bid gets his address compromised, he might avoid claiming the reward but the problem is the admin can initiate this claiming which leads to the winner loosing the reward

### Recommendation

The winner should have ability to change the address receiving the reward
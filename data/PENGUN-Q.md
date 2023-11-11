## [L] It is possible to cause DoS by placing many bids on Auction.

## Impact

It is possible to lock the NFT and the funds bid in the AuctionDemo.

## Proof of Concept

The claimAuction function is responsible for delivering the NFT to the winner and refunding the failed bids.

Since this function iterates over all the bids, a malicious user can spam bids to cause a DoS.

```solidity
contract Bidder {
    function callAuction(AuctionTest au) external payable {
        for(uint256 i =0;i<100;i++){
            au.participateToAuction{value: i+1}(1);
        }
    }
}
```

By creating a contract like the one above and calling it as soon as the Auction starts, the ETH required for bidding will be greatly reduced.

With many bids, the gas required for the claimAuction function increases, and if it exceeds the block gas limit, the claim will fail. If the users who participated in the bidding do not cancel before the endTime, they will not be able to withdraw their funds.

Since the bidding process requires more gas than the claim process, the attack is inefficient. However, due to the significant impact, it is recommended to modify the code to allow each participant to claim their own refund instead of relying on the winner to refund.

## [L] AuctionDemo.returnHighsetBidder does not update highBid and may return unintended values.

## Impact

The returnHighestBidder function does not compare highBid.

## Proof of Concept

```solidity
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

Although the if statement checks highBid, it does not update it afterwards, so highBid is always 0.

Therefore, the comparison is meaningless.

However, since the last active data in auctionInfoData is the highest bid, there may not be an immediate problem, but unintended results can occur in the future due to updates or other factors.

## [L] It is not possible to mint during allowlistStartTime if only public mint is supported.

## Impact

When selling a collection through public mint, it is not possible to purchase when block.timestamp is equal to allowlistStartTime.

## Proof of Concept

The getPrice function in the MinterContract returns the current price of the collection.

When calculating the price in this function, it uses allowlistStartTime as the reference time.

Therefore, if only public mint is supported, allowlistStartTime, allowlistEndTime, and publicStartTime must all be the same.

In this configuration, if you call mint when block.timestamp is equal to allowlistStartTime, the allowlist purchase logic will be executed due to the condition `if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime)`.

Since this collection only supports public mint, this logic will fail and revert will occur.

## [L] Problems may arise when using ARRNG.

## Proof of Concept

RandomizerRNG.sol is a contract that generates random hashes using ARRNG.

```solidity
function requestRandomWords(uint256 tokenid, uint256 _ethRequired) public payable {
        require(msg.sender == gencore);
        uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
        tokenToRequest[tokenid] = requestId;
        requestToToken[requestId] = tokenid;

    }

```

ARRNG provides the price required for random generation through an off-chain API.

```solidity
// Pass native token (e.g. ETH) for return gas. Excess is refunded.
<https://api.arrng.io/getCost?chain=1&quantity=1&complexity=3>
// chain Id e.g. 1 for ETH Mainnet
// quantity: number of numbers e.g. 1
// complexity: complexity of return function
//   1 = Simple, e.g. store a single var
//   2 = Medium, e.g. 2 to 3 writes>
//   3 = Complex. Lots of processing
// return: {"nativeToken":0.0129}

```

If you make a direct call to the above API, you can see that the price changes in real-time depending on the Ethereum gas price.

However, RandomizerRNG uses a fixed value called ethRequired in the requestRandomWords function.

Therefore, if the mainnet gas price rises sharply, the random request may fail.

In addition, to handle the cost of about 0.01 ETH based on complexity 2, a significant amount of funds must be present in the contract. If the funds run out, the mint process itself may be reverted.

## [L] When paying more ETH than the sale amount of the Collection, it does not refund the excess.

## Impact

Users who pay more ETH than the price are not refunded the remaining amount.

## Proof of Concept

NFT minting has a competitive nature, so there may be users who pay more ETH and participate in minting in order to increase the chances of success.

However, even if they are minted at a lower price than expected, they cannot receive the remaining amount.

## [L] NextGenCore.setCollectionData does not match the specification.

## Proof of Concept

The setCollectionData function has the following comment:

```solidity
// function to add/modify the additional data of a collection
// once a collection is created and total supply is set it cannot be changed
// only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set

```

First, there is no check for whether setFinalSupply has been set.

Also, _collectionArtistAddress and _maxCollectionPurchases can be changed, but _setFinalSupplyTimeAfterMint can also be modified.

```solidity
else if (artistSigned[_collectionID] == false) {
            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
        }

```

## [L] Using prevrandao can expose to MEV attacks.

## Proof of Concept

XRandoms.randomNumber generates a randomNumber using block.prevrandao.

```solidity
function randomNumber() public view returns (uint256){
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
        return randomNum;
    }

```

prevrandao is exposed by validators, and using it allows for MEV attacks.

## [NC] It is recommended to update the auction status in AuctionDemo.claimAuction.

## Impact

Even after claimAuction is completed, the auction status remains true.

## Proof of Concept

```solidity
function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
        auctionInfoData[_tokenid].push(newBid);
    }

```

The participateToAuction function checks the auction status in minterContract and requires it to be true in order to participate.

When claimAuction is executed, the NFT is transferred to the highest bidder and the auction ends, so it is recommended to change the auction status to false in claimAuction.
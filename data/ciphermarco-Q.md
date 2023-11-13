# NFTs Are Minted and Can Be Operated Before `tokenHash` is Set

This may have unwanted consequences in case an user finds a way to perform economical changes to the NFT. In one scenario, for example, the user could font-run the randomness fulfillment and sell or, if allowed, burn to mint into another collection.

Consider forbidding any user actions until the randomness request is fulfilled.

# Constructor Does Not Need `public` Modifier

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L36
```
constructor (address _minter, address _gencore, address _adminsContract) public {
```

# Approve Auction Contract

The Auction contract is not automatically approved to manage the NFT when put to auction. Not a big problem since it is already a trusted set-up managed by the team, but could be an oversight. Either way, it could make operatoins smoother.

# Redundant Logic in `auctionDemo`

This logic has some redundancies and is mostly overworked:

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L65-L83
```c++
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

Besides logic redundancy in the code itself, the whole concept of looping may be flawed. In the implemented auction architecture, the highest bidder will always be the last item pushed into `uctionInfoData[_tokenid]`. In any case, the `highestBidder` could be stored as a global variable for every new highest bid. Of course, there would have to be other changes if this last model was applied somehow.

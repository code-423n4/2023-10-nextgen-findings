### Unnecessary mappings tracking similar things
In RandomizerRNG:
```solidity
mapping(uint256 => uint256) public tokenToRequest;
mapping(uint256 => uint256) public requestToToken;
```
There’s no need to have tokenToRequest and requestToToken. You can just use one of these mapping. The current implementation is prone to inconsistencies and is just not needed.

In NextGenCore:
```solidity
mapping (uint256 => bool) private collectionFreeze;
```
Could just add a variable to the mapping `collectionAdditionalData` for example to see if it’s freezed.
```solidity
    // minted tokens per address per collection during public sale
    mapping (uint256 => mapping (address => uint256)) private tokensMintedPerAddress;

    // minted tokens per address per collection during allowlist
    mapping (uint256 => mapping (address => uint256)) private tokensMintedAllowlistAddress;

    // tokens airdrop per address per collection 
    mapping (uint256 => mapping (address => uint256)) private tokensAirdropPerAddress;
```
These could be combined as well, in all 3 you track how many minted/airdropped tokens per collection per address. You could create a struct with minted during allowlist, minted during public sales, and airdropped. And create a mapping with the struct.

### Function emergencyWithdraw() in RandomizerRNG is not following best practices

emergencyWithdraw doesn’t take any parameters and automatically sends all the balance to the adminsContract owner. 

`address admin = adminsContract.owner();` 

However, it’d be much better if you could specify to which parameter the funds need to be sent in case an owner is compromised or simply for convenience.

### Change name from auctionDemo to AuctionDemo

It is a good practice to keep the name consistent with the contract file name.

### In AuctionDemo - refunding all bidders in claimAuction() is very bad practice

Bidders only get refunded when the Winner or an admin claims the auction. This is very bad practice due to numerous reasons:

- The function can become very gas-expensive and revert due to an out-of-gas error if there are too many bidders in an auction
- Bidders depend on the winner or admin to receive their funds back which could be annoying
- A malicious bidder can DOS the entire function making the Winner unable to receive his reward and the other bidders not receive their funds back

Suggestion: Implement a “pull” pattern - for example, a `refundBid` function where each bidder can claim their refund after the auction has ended. Or simply refund the bidders and remove them from the array once they are not the Highest bidder since only the Highest bidder wins anyway.

### Add emergencyWithdraw function to AuctionDemo

There could be cases where ETH gets stuck in the AuctionDemo contract - for example if  DOS is being done on the `claimAuction` function as described in a submitted issue, or simply if some bidder can’t accept his reward or refund(not implemented receive function or his wallet got exploited).

### In AuctionDemo - claimAuction sends the funds from the auctioned NFT to the contract owner instead of the NFT owner

In line `(bool success, ) = payable(owner()).call{value: highestBid}("");` we’re sending the `highestBid` to the owner of the contract although it should be sent to the seller of the auctioned NFT.

### In AuctionDemo.sol the constructor is explicitly set as public

There’s no need to do that
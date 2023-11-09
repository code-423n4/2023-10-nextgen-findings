## Auction bids don't need to be a % percentage bigger than the previous one
Link: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L58
In AuctionDemo.sol we can place bids in an auction by calling `participateToAuction`. This function requires that `msg.value > returnHighestBid`. This means that any new bids which is bigger than the previous one will be accepted, even if the bid is only 1 wei bigger. This approach doesn't seem to lead to big problems, but in my opinion this is an unfair system for the users. I think that for being able to place a bid you should consider forcing the new bid to be at least a % bigger than the current biggest bid. For example: if the biggest bid right now is 1 ETH, for placing a bid users need to send at least 1.01 ETH, making it 1% bigger than the last bid.

## `returnHighestBidder` doesn't update `highBid`
Link to `returnHighestBidder`: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87-L100
This function is for getting the highest bidder in a certain auction. As we can see, it iterates through the array and checks if the bid done is bigger than `highBid`. This check will always pass since `highBid` is not updated and it's value is always 0. This doesn't represent bigger problems since the array is already sorted and the `status` is checked for getting the index with the biggest bid.

## Auction won't be claimable if the owner of the NFT doesn't approve the AuctionDemo.sol contract
Link: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L112
When an auction is claimed using `claimAuction` the NFT is transferred from the owner of the NFT directly to the highest bidder. Even if the address that owns that NFT is trusted, I think a better approach would be sending the NFT directly to the AuctionDemo.sol contract when using `mintAndAuction` in MinterContract.sol, instead of sending it to a trusted address.

## No check of call success in ETH transfers
Some examples:
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L128 
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L139
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L113
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L116

Even if this issue was already in the automated findings, I think it's worth explaining the impact this could have in this specific context. 

Users get their ETH refunded when not winning an auction. Also, they can cancel their bids before an auction is finished and get their ETH refunded. Not checking that the ETH was correctly transferred from `AuctionDemo.sol` to the users will result in this ETH getting stuck in the contract, since there is no possibility of withdrawing it in any other way. This happens because even if the ETH transfer doesn't work the transaction will still succeed. In the case of cancelling a bid, the `status` of the bid is still updated and users won't be able to call the function again. In the case of claiming an auction, we have a similar case, since `auctionClaim` will still be updated.

I want to remark that there is no way of getting the stuck ETH back even for the admins, because there are no functions like an `emergencyWithdraw` or similar. This will result in a loss of funds for the users, having their ETH stuck in the contract.

For fixing this issue we should check that `success` is true after making the ETH transfer using the `call`.    

## The `maxCollectionPurchases` for the collections can be bypassed by minting with another wallet
Link: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L224
This check isn't really doing anything, since the same user could just use another wallet for minting more NFTs. There is no possible way of checking how many NFTs each user buys. In my opinion, this check will hurt the users and the protocol. The reason is that "normal" and unexperienced users wouldn't think about changing wallets for buying more NFTs, and more experienced users would take advantage of this. If there is a `maxCollectionPurchases == 1` in a collection with 1000 NFTs, users would think that there are 1000 holders. This is in fact not true, since an experiences user could use a script or manually buy 500 NFTs by simply using different wallets. In my opinion it is an unnecessary check, because you will only limit the unexperienced users.

## No check if too much msg.value is sent when minting NFTs
Example:
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L233

Even if this issue was already in the automated findings, I think it's worth explaining the impact this could have in this specific context. 

As there is no function for withdrawing excess funds in `MinterContract.sol`, all these funds will get stuck forever in the contract. I would advise to change the check from `msg.value >= getPrice()` to `msg.value == getPrice()` to prevent any lose of funds.

## Use Basis Points for percentages 
Links:
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L370-L371
- https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L429-L433

In order to have better precision when calculating the funds to be sent to the artists, I would suggest using Basis Points for assigning the percentages. For instance, 100% would be represented as 10_000. This will prevent possible rounding errors and less funds being distributed. This won't be a problem when there is a lot of ETH to be sent, but for example if there is only 1 ETH to be distributed across the 5 wallets, with some of them having very low percentages, this could be a bigger issue.

## A collection's randomizer can be changed even if the collections is frozen
Link: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L170-L174
Even if a collection has been frozen, it's randomizer can still be changed. Even if this function would be called by a trusted role, I think it shouldn't be possible. Freezing a collection gives users the certainty that collection's importan info, etc. won't be changed. Nonetheless, this will entail users' getting different token hashes than they thought.

## A collection's randomizer can be changed even if the collections is frozen
Link: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L170-L174
Even if a collection has been frozen, it's randomizer can still be changed. Even if this function would be called by a trusted role, I think it shouldn't be possible. Freezing a collection gives users the certainty that collection's importan info, etc. won't be changed. Nonetheless, this will entail users' getting different token hashes than they thought.

## Request confirmations within Chainlink's VRFv2 should be bigger
Link: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L28
Even if the value returned from Chainlink's VRFv2 won't have a big impact in the particular context of this project, it is advised to use a value which is bigger than the common length of the block reorgs that can happen in the particular chain. As this project is going to be deployed only in Ethereum Mainnet, changing it to `requestConfirmations = 10` would be more than enough. Reorgs have already happened in Ethereum. One of the biggest ones was 7 blocks, not so long ago.
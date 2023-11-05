### Overview of the Protocol

- The NextGen Protocol helps artists to mint an NFT collection. All NFT created by NextGen will be part of the NextGen Collection.
- The artist can create up to 10 billion NFTs per collection
- The protocol supports different types of minting fee, such as fixed fee, linear/exponential descending fee and fixed percentage ascending fee.
- The protocol takes a percentage cut from every NFT sale.
- There are some unique functions that NextGen has. Firstly, a user can burn an NFT created through NextGen to mint an NFT in a NextGen Collection through the burnToMint() function.
- A user can also burn an external NFT (subject to protocol's approval) to mint an NFT in a NextGen Collection through burnOrSwapExternalToMint().
- NFTs can also be minted and auctioned by the admin using the mintAndAuction() function and the AuctionDemo contract.

### Walkthrough of the Protocol

- As an artist, the first step is for the artist's team to get in touch with the protocol team.
- The protocol will call createCollection() in NextGenCore.sol, and provide the necessary details like collection name, description, etc.
- The protocol will then register a collection admin from the artist team.
- The collection admin will then call setCollectionData() and provide more details on the collection like total supply, max collection purchase limit.
- After setting the collection data, the collection admin can set the collection cost and collection phases in the MinterContract, which includes the mint price, sales option, mint duration.
- At the same time, the protocol admin will set the primary and secondary splits and the percentages. This indicates how much percentage the team will earn for every minted NFT.
- Once timing and costs has been set, users can start to interact with the protocol and call mint().
- The money earned from the NFT sales will be stored in the MinterContract. To retrieve the money, payArtist() is called, and the payment is distributed to the artist team and the protocol team respectively.
- Once the mint duration is up, the admin can call setFinalSupply to set the final supply of the collection

To create another collection, the whole process will restart.

### Codebase Quality Analysis and Review

| Emoji                   | Description                            |
| ----------------------- | -------------------------------------- |
| :white_check_mark:      | Fully Checked, Working as Intended     |
| :ballot_box_with_check: | Fully Checked, Requires Slight Changes |
| :red_circle:            | Checked, Something seems off           |

| Contract       | Function                                | Explanation                                 | Comments                                                                         | Coverage                |
| -------------- | --------------------------------------- | ------------------------------------------- | -------------------------------------------------------------------------------- | ----------------------- |
| MinterContract | setCollectionCosts                      | Set a Collection Minting Cost               | CollectionAdmin, Once set, cannot be changed, collection must be created already | :white_check_mark:      |
| MinterContract | setCollectionPhases                     | Set a Collection Start/End Time             | CollectionAdmin, Timings are not checked against each other                      | :ballot_box_with_check: |
| MinterContract | airDropTokens                           | Mints a free NFT to a recipient             | Admin, Collection must be created first, checks total limit correctly            | :white_check_mark:      |
| MinterContract | mint                                    | Mints an NFT to a recipient                 | Payable, users do not get refund, 2 phases and 3 salesOption                     | :ballot_box_with_check: |
| MinterContract | burnToMint                              | Mints an NFT to a recipient                 | Payable, initializeBurn must be set to true, only public access, mints 1 only    | :ballot_box_with_check: |
| MinterContract | mintAndAuction                          | Mints an NFT to a recipient, set Auction    | Admin, mints one NFT per time period, collectionTokenMintIndex repeated          | :white_check_mark:      |
| MinterContract | initializeBurn                          | Initialize burn to mint                     | Admin, might be a hassle if there's a lot of users wanting to burnToMint         | :white_check_mark:      |
| MinterContract | initializeExternalBurnOrSwap            | Initialize burnOrSwapExternalToMint         | Admin, can swap external NFT for a nextGen NFT                                   | :white_check_mark:      |
| MinterContract | burnOrSwapExternalToMint                | Swap external NFT for nextGen NFT           | Admin, external NFT will be sent to burnOrSwapAddress                            | :white_check_mark:      |
| MinterContract | setPrimaryAndSecondarySplits            | Set primary splits                          | Admin                                                                            | :white_check_mark:      |
| MinterContract | proposePrimaryAddressesAndPercentages   | Propose primary addresses and percentages   | Artist/Admin                                                                     | :white_check_mark:      |
| MinterContract | proposeSecondaryAddressesAndPercentages | Propose secondary addresses and percentages | Artist/Admin, unclear of usage in protocol                                       | :ballot_box_with_check: |
| MinterContract | acceptAddressesAndPercentages           | Accept primary addresses and percentages    | Admin                                                                            | :white_check_mark:      |
| MinterContract | payArtist                               | Pay artist and protocol team                | Admin, success not checked                                                       | :ballot_box_with_check: |
| MinterContract | emergencyWithdraw                       | Withdraw any balance from the contract      | Admin, huge centralization (able to withdraw all earnings), success not checked  | :ballot_box_with_check: |
| MinterContract | getPrice                                | Get the minting price of collection         | Different types of price, descending, ascending, fixed                           | :white_check_mark:      |
| NextGenCore    | createCollection                        | Create a Collection                         | Admin                                                                            | :white_check_mark:      |
| NextGenCore    | setCollectionData                       | Set collection data                         | CollectionAdmin                                                                  | :white_check_mark:      |
| NextGenCore    | airDropTokens                           | Airdrops 1 NFT to a recipient               | OnlyMinter, airdrops 1 NFT to recipient, supply check works as intended          | :white_check_mark:      |
| NextGenCore    | mint                                    | Mint an NFT                                 | OnlyMinter, depends on phase, supply check works as intended                     | :white_check_mark:      |
| NextGenCore    | burn                                    | Burns an NFT                                | Public (not sure if intended, requires approval), increases the burn amount      | :ballot_box_with_check: |
| NextGenCore    | burnToMint                              | Burns an NFT to Mint an NFT                 | OnlyMinter                                                                       | :white_check_mark:      |
| NextGenCore    | updateCollectionInfo                    | Update Collection Info                      | CollectionAdmin, change collectionInfo, depends on index                         | :white_check_mark:      |
| NextGenCore    | artistSignature                         | Signs a collection                          | CollectionArtistAddress, signs a collection                                      | :white_check_mark:      |
| NextGenCore    | freezeCollection                        | Freeze a collection                         | Admin, frozen collection cannot update image, collection, change token data      | :white_check_mark:      |
| AuctionDemo    | participateToAuction                    | Participate in an Auction                   | before auction end time, must be higher than highest bid                         | :white_check_mark:      |
| AuctionDemo    | returnHighestBid                        | Returns the highest bid                     | If highest bid is withdrawn, the second highest will become highest bid          | :white_check_mark:      |
| AuctionDemo    | returnHighestBidder                     | Returns the highest bidder                  | Same as highest bid, returns an address                                          | :white_check_mark:      |
| AuctionDemo    | claimAuction                            | Claim the NFT after an Auction              | Winner/Admin, claims the NFT, refunds all participants, some M issues reported   | :ballot_box_with_check: |
| AuctionDemo    | cancelBid                               | Cancel a bid                                | Refunds a bid, must be before auction end time                                   | :white_check_mark:      |
| AuctionDemo    | cancelAllBids                           | Cancel all bids                             | Cancel and refunds all bids                                                      | :white_check_mark:      |

##### MinterContract.sol (Deep Dive)

**mint()**

- :white_check_mark: phase 1, only allow list, have delegator (delegation management contract OOS)
- :white_check_mark: phase 1, only allow list, no delegator (delegation management contract OOS)
- :white_check_mark: phase 2, public access
- viewTokensIndexMin(col) is a fixed number
- If salesOption == 3, only can mint 1 token.
- Allowlist needs a merkleProof
- tDiff = number of periods that has passed

**getPrice()**

- :white_check_mark: salesOption == 3, with rate set (price will increase every mint, rate set in percentage)
- :white_check_mark: salesOption == 3, without rate set(price will stay the same every mint, rate set in percentage)

```
        if (collectionPhases[_collectionId].salesOption == 3) {
            // increase minting price by mintcost / collectionPhases[_collectionId].rate every mint (1mint/period)
            // to get the price rate needs to be set
            if (collectionPhases[_collectionId].rate > 0) {
                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
            } else {
                return collectionPhases[_collectionId].collectionMintCost;
```

- :white_check_mark:salesOption != 3 or 2 (price will stay the same every mint)

```
        } else {
            // fixed price
            return collectionPhases[_collectionId].collectionMintCost;
```

- :white_check_mark:salesOption == 2, without rate set (exponential decrease until resting price, rate set in wei)
- checked with timePeriod = 1 hour, mint price = 0.5e18 ETH
- If time passed = 1 min, tdiff = 0, decreaserate = 0.25e18 / 1 hours \* 1min = 0.00416, price = 0.5e18 - 0.00416e18 = 0.4958e18
- If time passed = 30 min, tdiff = 0, decreaserate = 0.25e18 / 1 hours \* 30min = 0.125e18, price = 0.5e18 - 0.125e18 = 0.375e18
- If time passed = 50 min, tdiff = 0, decreaserate = 0.25e18 / 1 hours \* 50min = 0.208e18, price = 0.5e18 - 0.208e18 = 0.292e18
- If time passed = 1 hours, tdiff = 1, decreaserate = 0, price = 0.25e18
- If time passed = 1.5 hours, tdiff = 1, decreaserate = (0.25e18 - (0.5e18/3)) / 3600 \* 1800 = 4.16e16, price = 0.25e18 - 4.16e16 = 0.208e18
- If time passed = 2 hours, tdiff = 2, decreaserate = 0, price = 0.5 / 3 = 0.16666e18

Seems like the exponential descending calculation is correct

- :white_check_mark:salesOption == 2, rate set (linear decrease until resting price, rate set in wei)
- checked with timePeriod = 1 hour, mint price = 0.5e18 eth, min price = 0.1e18 eth, rate is 1e17
- If time passed = 30 mins, tdiff = 0, 4 > 0, price = 0.5e18 eth - (0 \* 1e17) = 0.5e18 eth
- If time passed = 1 hours, tdiff = 1, 4 > 1, price = 0.5e18 eth - (1e17) = 0.4e18 eth
- If time passed = 3.5 hours, tdiff = 3, 4 > 3, price = 0.5e18 eth - 3e17 = 0.2e18 eth
- If time passed = 5 hours, tdiff = 5, 4 < 5, else clause, price = collectionMintCost = 0.1e18 eth

Seems like the linear descending calculation is correct

##### Preliminary Questions

MinterContract

1. Is the admin of ArtistOrAdminRequired and FunctionAdminRequired the same?
2. Why does proposePrimaryAddressesAndPercentages and proposeSecondaryAddressesAndPercentages have 3 addresses specifically?
3. What is the point of proposeSecondaryAddressesAndPercentages if it's not used in payArtist?
4. Why is the mint function so long? What is the function trying to accomplish? Line 196
5. Whats the difference between salesOption == 3 and salesOption == 2 and salesOption == (other number)? Line 532 onwards
6. What is phase 1 and phase 2? Line 203, 222
7. What is airdropTokens?
8. What does burnToMint mean?

##### Preliminary Answers

Minter Contract

1. Yes, they are the same.
2. Probably nothing, just design.
3. Not sure.
4. The mint function distinguishes between allowlist timings and public timings, and whether the token minted has a delegator.

- Looking at the code, from Line 202 to 220 in the minter contract, the mint function focuses on the allowlist users only.
- The first if statement checks that the caller sets a delegator (Line 205), and the else statement means that the caller is minting for himself (Line 215).
- Inside both if and else statements, `_maxAllowance` is checked, calling the `gencore.retrieveTokensMintedALPerAddress`, making sure that the allowlisted address do not mint past the max allowance.
- collectionTokenMintIndex is checked. (checked, it tallies up)
- Mint is called in gencore contract. Circulation supply is increased and checked against total supply. (checked, it tallies up)
- \_mintProcessing is called which invokes ERC721
- If salesOption == 3, only 1 mint per time period (checked, tallies up when timePeriod = 1 days (86400 seconds))
- (timeOfLastMint will be 1 day before allowliststart, the first mint of a collection is allowed)
- (lastMintDate will then update to the the time of allowliststart, the second mint can only take place 1 day after)
- salesOption == 3 means that the price will be linear, one NFT will be more expensive than the next
- salesOption == 2, means price decreasing exponentially or linearly, depending on whether rate is set (linearly = rate is set)
- salesOption == other, means fixed NFT price

5. The difference salesOption indicates the pricing of NFT

- salesOption == (other number) means that the NFT will always be at a fixed price
- salesOption == 3 means only 1 mint allowed every time. If rate is set, the mint price will go up linearly. Rate here indicates the percentage. If rate is not set, the nft will be at fixed price
- salesOption == 2 means the NFT price will decrease until a minimum price. If rate is set, the price will decrease exponentially. If rate is not set, the price will decrease linearly. Rate is stated in wei.

6. Phase 1 is allowlist access and phase 2 is public access. Phase 1 must be earlier than phase 2.
7. AirdropTokens can only be used by the admin, it gives an array of recipient one NFT each for free (recipients don't have to pay any money)
8. burnToMint means burning an older token to mint a new token. From the code, mintIndex is the index to be minted, and \_tokenId is the token to be burned. Users can burn an NFT in their nextGen collection and mint another NFT in the nextGen collection, provided they are given permission by the admin.

### Centralization Risks

| Contract       | Actor           | Description                                                           | Severity | Reason                                      |
| -------------- | --------------- | --------------------------------------------------------------------- | -------- | ------------------------------------------- |
| MinterContract | FunctionAdmin   | Controls the updates, artists and team splits, and emergency withdraw | Critical | Able to withdraw all earnings               |
| MinterContract | CollectionAdmin | Sets the collection costs and phase                                   | Low      | Artist team                                 |
| MinterContract | Artist          | Propose Percentages                                                   | Low      | Artist team                                 |
| NextGenCore    | FunctionAdmin   | Creates Collection , freeze collection, set royalties                 | Medium   | Able to set royalties and freeze collection |
| NextGenCore    | CollectionAdmin | Update collection info, change metadata                               | Low      | Artist team                                 |
| NextGenCore    | MinterContract  | Interacts with MinterContract to mint, burn etc                       | None     | Contract                                    |
| AuctionDemo    | Admin           | Call claimAuction                                                     | Low      | Intended Design                             |
| AuctionDemo    | Winner          | Call claimAuction                                                     | Low      | Intended Design                             |

### Architecture Review

##### Pros of using NextGen

- Convenient for the artist. Most of the minting process is covered by the protocol team
- If a user have a NextGen collection NFT, he can burn it for another NextGen NFT (with admin's approval). This can introduce a tier system for NFTs.
- Clear payment procedure and royalty splits.
- Lots of room to create unique, generative NFTs

##### Cons of using NextGen

- User can only pay in native token
- The admin has a extremely huge control over the process, including payments. There must be a lot of trust in NextGen Protocol when creating a collection on the platform.

### Approach, Comments and Learnings

- Thoroughly auditied the whole scope, except for the delegation, merkle proof and strings part.
- Interesting to see how a exponential descending price is coded.
- Protocol also doesn't use external imports like OpenZeppelin but instead copied all the contracts. Pretty interesting style, simple and convenient, but may lack upgrade capabilities.
- The NextGen protocol can house many collections without any Id collisions.

### Time spent:
40 hours
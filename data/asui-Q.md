1. ```cancelAllBids``` function only cancels the msg.sender's bid even though the name suggest to cancel all bids.
   Functions like these should never even be used because if we were to cancel all the bids at once and since we are dealing with .call with values as eth they can be reverted and DOS by a malicious address. 

2. In the minter contract in ```mintAndAuction``` function should check that the given ```_auctionEndTime``` is greater than the current block.timestamp. Otherwise if there is an auction with an already expired auction end time it makes no sense.

3. There are multiple functions with loops in the auction contract which can cost too much gas and revert or revert with external calls because of a call to a malicious address. Especially the ```claimAuction``` function where event the modifier calls a loop. To avoid these make the functions so that the users can only take out their bids and remove the loops and use mappings to directly get the user's bid info instead of looping through all the infos.

4. In the MinterContract in ```setCollectionPhases``` function add a check to make sure that the allow list minting period and the public minting period doesn't overlap. Otherwise if incase they overlap when users call mint they will go through the first if statement which is the allow list minting period and cannot mint until the allow list period is over even though the public minting period has started.

5. When setting a collection cost, for mintpass tokens costs can be set to 0 so make sure that the collection's randomizer is the ```NextGenRandomizerNXT``` contract otherwise the protocol's LOOKS tokens(with chainlink vrf) and eth(for arrng) can be used up or drained for tokens with 0 values. 

6. When minting a token the ```_saltfun_o``` can be removed form the ```airDropTokens```, ```mint```, ```burnToMint```, ```_mintProcessing``` and in the ```calculateTokenHash``` function in all the randomizer contracts since they are never even used.  
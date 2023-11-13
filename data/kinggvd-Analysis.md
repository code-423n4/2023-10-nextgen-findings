Context:
Hi,
this is my first audit, so I hope my analysis is not off. I focused more on the business logic aspects than the Web3 specific ones, since I have only started learning about those.

My approach was to first analyze the overall code structure and syntactic errors. I then focused on the business logic layer and afterwards on Web3 specific stuff.

Nextgens architecture could greatly be improved by using more OpenZeppelin contracts which provide best practices for Role-Management (General-Admin, Function-Admin, etc), common attack vectors (reentrancy) and others.

Additionally, the architecture should be based more around producing events (participateInAuction, cancelBid, claimAuction) and withdrawing the money in separate logic. Ensuring the risks explained in the analysis are not exploitable.

Furthermore, the whole bidding system needs to be reworked since it can easily allow bad actors to win any bid as explained below. There should be a system in place to mitigate this. (Also drafted below)

Please feel free to provide me with any feedback on the analysis itself. Unfortunately, I only had limited time to write and submit my report due to problems during the registration-process.

Kind Regards,
kinggvd


Analysis:
Major:
1. AuctionDemo.sol participateToAuction
    * bidding is only allowed if the bid is higher than current highest_bid but there are no checks in place to stop an actor from blocking others bids by providing a high value which other bidders are not ready to outbid. The actor can then remove this bid at last the minute and bid a much lower value winning the bid and producing no added value for Nextgen. Either still allow and maintain all bids ensuring that lower values are considered or put restrictions on cancelBid e.g. (time-limit, cancellation-fee) new higher bids could also cancel older lower bids of the same address, etc. There are many solutions to this problem with their own advantages and disadvantages
2. MinterContract.sol claimAuction and cancelBid/cancelAllBids
    * these functions could be called at practically the same time and transfer the token while refunding the bid. A more separated and withdraw based solution for canceling/claiming should be used to ensure this cannot happen.
3. MinterContract.sol and RandomizerRNG.sol emergencyWithdraw
    * This can easily be abused by malicious actors to drain all funds
4. MinterContract.sol getPrice:
    * This function never produces a non-zero decreaserate and thus forces minting at a higher value
    * decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
    * price = p; block.timestamp = n; collectionPhases[_collectionId].collectionMintCost = c; Diff = d; collectionPhases[_collectionId].timePeriod = f; collectionPhases[_collectionId].allowlistStartTime = s
    * decreaserate = ((p-c/(d+2) / f) * (n - (d*f) -s)
    * the right side of this multiplication is defined as: (block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime)
    * or right-hand-side = n - (d*f) -s
    * tDiff is defined as: d = (n - s)*f
    * n - d*f - s if we insert d here and simplify we get:
n - ((n-s)/f) * f - s
n - (n-s) - s
right-hand-side = n-n+s-s
right-hand-side = 0+0 = 0
decreaserate = ((p-c/(d+2) / f) * 0
    * thus the decreaserate always yields 0




Minor: 
1. All arrays are basically unbounded, which can be used for DOS attacks and increasing gas usage. Old bids could simply be removed or some form of rate-limiting introduced
2. block.timestamp < minter.getAuctionEndTime checks are not consistent. Change this to avoid cancelation and refund while claiming (unlikely)
    * claimAuction: block.timestamp >= minter.getAuctionEndTime. replace >= by >
    * cancelBid and cancelAllBids: block.timestamp <= minter.getAuctionEndTime. replace <= by <
3. AuctionDemo.sol participateToAuction: initialize auctionClaim[_tokenid] = false
4. MinterContract.sol getPrice: else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){ // use >= <= for time check
5. AuctionDemo.sol returnHighestBidder: if (auctionInfoData[_tokenid][index].status == true)  should add length > 0 check here, to avoid any index errors with returnHighestBidder and WinnerOrAdminRequired; This also has to do with using Errors instead of revert(SOME_STRING) in order to produce more maintainable errors


### Time spent:
3 hours
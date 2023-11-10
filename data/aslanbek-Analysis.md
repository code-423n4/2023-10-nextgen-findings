# Recommendations 

## Contracts are too close to the 24kB limit
Contracts' size is almost too big for Ethereum Mainnet. Since it is explicitly stated that there will be additional functionality (judging by `saltfun`'s description), and since there may be mitigations that would increase contracts' size, it is advised to split the contracts into smaller ones, particularly NextGenCore and MinterContract.

## Auction design suggestions

### Increase auction deadline with every bid

Current auction design has a constant time of ending. This may disincentivize many users from participating, as tech-savvy users get an unfair advantage: their response time can be instantaneous, and they can pick just the right bid at the last block of the auction. Bumping the deadline with every bid will create a better experience for the bidders and increase artists' gains as a consequence.

### Use mappings instead of dynamic arrays

Current design, [as it was as well reported by the bot](https://github.com/code-423n4/2023-10-nextgen/blob/main/bot-report.md#h-01-permanent-dos-due-to-non-shrinking-array-usage-in-an-unbounded-loop), is vulnerable to permanent DoS. The issue can be properly mitigated via mappings, but will require disabling `cancelBid` for the highest bidder (because the second highest bid will need to be stored, then the third one, etc. - and it would require dynamic array again).

Consequently, the refunds will have to be claimed manually instead of during claimAuction, which will increase the total gas consumption, but decrease it for the auction winner (if he does not want to wait for the admin to call it).

## MinterContract#mint allows the delegate to choose an arbitrary receiver of an NFT

Although it could be said that the delegate is trusted, there's no reason to give them more permissions than they really need. Consider checking `delegator == mintTo`, if the mint is done by a delegate.


### Time spent:
40 hours
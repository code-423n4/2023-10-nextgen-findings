# Recommendations 

## Contracts are too close to the 24kB limit
Contracts' size is almost too big for Ethereum Mainnet. Especially with proposed mitigations, and since it is explicitly stated that there will be additional functionality (saltfun parameter), it is advised to split the contracts into smaller ones.

## Auction design should be reworked

Current auction design has a static time of ending. It gives unfair advantage to tech-savvy users, as their response time can be instantaneous, and they can pick just the right bid at the last block of the auction.

Recommendation - increase the deadline of the auction every time a bid is made

## MinterContract#mint allows the delegate to chose an arbitrary receiver of an NFT

Although it could be said that the delegate is trusted, there's no reason to give them more freedom than they need. Consider checking delegator == mintTo, if the mint is done by a delegate.



### Time spent:
40 hours
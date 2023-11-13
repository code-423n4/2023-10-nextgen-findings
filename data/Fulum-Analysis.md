## General

I audit the NextGen project during 1 week. Previously not audited and I expected to finds some interesting vulnerabilities.

## Approach

Most the focus was on a deep understanding of the sales models and the differents ways of minting a token. Verify the correctness of the functions, computations, transfer. Also focused on the generation of the randomness with the differents methods.

## Architecture recommendations

Architecture is classic and simple, it's easy to understand how the protocol work and it's a good thing.
I recommend the following architectural improvements:

- Recommend to add a pause feature in the important contract/functions to prevent from worst scenarios.

- Reinforce your architecture in creating special numbers for each sales models, choose one and implement the good logic in the setup of the collections and phases of minting. It helps you to not make mistake on collections/minting phases creations, have a more clear vision of the sales models and an esaye way to add new sales models in th future.

- Implement properly the randomness generation with the ChainLink VRF, the Chainlink documentation recommend for example to minting the token on randomness fulfillment to avoid potential revert of the request or multiples requests for the same token.
Chekc this documentation:
https://docs.chain.link/vrf/v2/security
https://docs.chain.link/vrf/v2/subscription#request-and-receive-data
https://docs.chain.link/vrf/v2/subscription#subscription-limits

- You can improve and simplify the `AuctionDemo` contract by deleting the power to `cancelBid()` for a user. This allows you to register the highest bidder when a new one coming, and to remove the for loops on the contract which present Denial-of-Service risks. Provide way to user to withdraw bids when auction is ended, help the user to withdrawn even if the `claimAuction()` function is DoS
I recommend to look at this auction contract architecture: https://github.com/stader-labs/ethx/blob/mainnet_V0/contracts/Auction.sol

## Qualitative analysis

Documentation is writed in simple and good way. You can add a category for security. Some functions explanations are missing from the documentation. 
The code is well writed but some mistakes and inadvertent error were made on the contracts lead to calculation errors, logic errors, problems of timing in the differents phases of the minting process and errors in randomness creation.
Recommend you to add NatSpec in your contracts.

## Centralization risks

The Admin contract (`NextGenAdmins`) is a good idea to manage all the actors of the ecosystem. The team use a multisig wallet and will be the owner of the admin contract to call the admin functions and add/remove privilege of differents actors. 
The problem is the power of the admin is too high and lead to potential rug pull scenarios like stealing funds or not paying artist. 
Example, admin have a lot of power for changing parameters of a collection in each time and broke important process. You can thinking about a way to have less power in sensitive time to guarantee the security of the users. It reinforces user confidence in the protocol.
Also need to give artists the power to claim their wages.

## Systemic risks

The primary systemic is the contracts unupgradability, in case of a security problem (epecially in the NextGenCore), it will be challenging to change logic. Thinking about potentials worst scenarios and implement functionnality (like upgradability, pause functions) and scenario to react quickly in case of issue.

Another risk is calling external contract which can be untrusted with a low level call, add risk of re-entrancy. Also a `safeTransferFrom` method to an external ERC721 collection which add risk of re-entrancy and/or no transferring token.

MEV is a systemic risk, especially on the auction system with block stuffing attack and front-running attack (example from preventing a user to give a higher bid).

## Time Spent

5 days 

### Time spent:
50 hours
# Analysis of the NextGen protocol

### Access Control

The protocol uses a rather unique approach for access control. It includes the conventional role-based one with the following 4 roles: `Owner`, `Global Admin`, `Collection Admin`, and `Function Admin`. The `Function Admin` role is the most interesting one here. It allows `Owners` and `Global Admins` to grant tailored access to specific functions to users. This is how a significant portion of the "write" functionality is protected within the protocol.

All of this is managed through a primary contract called `NextGenAdmins`. All contracts throughout the protocol interact with this main access control contract to determine users' permissions, eliminating the need for redundant checks in all contracts.

While reviewing it, I couldn't find any way to gain more control than what my permissions allow, or to otherwise compromise the system. It appears to be working well for the protocol's intended use case.

### Centralization

The protocol is quite unusual in this aspect. While, in the most common sense of the term, it can be considered highly centralized, it's important to note that the protocol is primarily interacted with by trusted roles. `Artists`, despite being considered trusted parties, have the least amount of privileges. When it comes to creating a collection or minting a token, a significant amount of off-chain communication has to occur between the protocol team and the artists to initiate the process, again, because of artists' default privileges. Despite this, they are granted some on-chain access to modify data once it is created. For instance, they can set or update collection data and provide their signature, in addition to having apparent read-only access. The only contract that seems to be intended for all kinds of external users is the `AuctionDemo`.

### Conventions/Style

While reviewing the protocol, I noticed that some of the most basic Solidity conventions are not followed. For instance, there are contract names using camel case, modifiers using pascal case, structs using camel case, various unneeded checks like "== true" and "== false", etc.. It is recommended that the protocol developers read the official [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html) and follow it.

There are also very good comments provided for each function in the documentation but for some reason, they're not present in the code. Rather, there are simple one-line comments like "mint function" and "airdrop function" that do not provide any meaningful value to the reader.

There are also some noticeably large functions within `MinterContract` like `mint`. They could've been easily split up. In the case of `mint`, the delegation, Merkle proof, and price calculations logics could go into separate functions/modifiers.

The code could also be tightened up by adding more `require` checks but it's not really that bad and I couldn't find a case where a wrong input without validation could cause any irreversible damage.

### AuctionDemo contract

`AuctionDemo` is probably the contract I liked the least. It seemed like it was rushed or not thoroughly developed/thought out. I discovered some vulnerabilities in it and those could've been easily avoided by implementing pull-over-push patterns. The contract is also not tested, or at least the tests were not included in the repository for this contest.

The other weird aspect was the logic. It doesn't resemble a normal auction. For instance, let's say Alice bids 10 ether for an NFT (I'll use ether only to keep it simple), the Bob comes in and bids 11 ether. Alice sees that she's been outbid so she wants to increase her bid. Now, instead of bidding 1 ether on top of her initial bid, she would have to bid 12. For some reason, bids work like this and don't accumulate. There's logic that allows each individual bid of a particular bidder to be canceled. When it is canceled, the ether is given back to the bidder, but then again, this logic seems weird, cumbersome, and unnecessary.

The current structure employs arrays for storing all bids. This involves using loops in various functions to query the data. As the data increases, additional gas costs are also incurred.

The protocol team should consider rewriting this contract altogether and employ a more robust approach avoiding the use of arrays and loops.

### Time spent:
40 hours
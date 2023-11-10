# Analysis Report
## Approach
The approach taken in this audit was to first check the contract's Chainlink VRF functionality, then check the interactions between `MinterContract.sol` and `NextGenCore.sol`, and then review the rest of the codebase.
## Architecture
The architecture of the protocol is generally good. Specifically, the ability to change the randomizer contract's address is a very valuable feature in case of a compromised randomizer contract or compromised randomness oracle. This design choice does introduce some centralization risk, however. Despite good design choices like this, the protocol's architecture has a flaw in that `MinterContract` and `NextGenCore` have many similar functions and both store state data for collections. This may be risky because it introduces extra complexity to the protocol. Furthermore, the collections data may become inconsistent between `MinterContract` and `NextGenCore`. I did not have time to look deeper into this area; invariant testing may be very helpful to uncover issues here. It may be better to combine `MinterContract` and `NextGenCore` into a single contract.
## Centralization Risks
The protocol admin can perform a large variety of actions, so there is a very high degree of centralization risk. This should be improved by limiting the actions of trusted roles, and/or delegating privileged functionalities to different roles wherever possible. For example, the protocol admin can change collection minting costs; this functionality may be better suited to be delegated to the artist, to reduce the risk of a compromised admin griefing collections. Similarly, artists should be able to claim their royalties without having to wait for the admin to do it for them. 

Also, it is essential that different privileged roles are assigned to different addresses/people; this way if one role is compromised, other non-compromised roles can mitigate the damage done to the protocol.
## Other Considerations
Adherence to best practices in the codebase can be improved. The CEI pattern is generally followed, but adding reentrancy locks would also reduce risk by reducing attack surface area. Randomizer addresses should be set when initializing collections; minting when the randomizer hasn't been set will error without a protocol-specified reason.

The AuctionDemo contract currently uses a O(n) algorithm (iterating through an array) to lookup the highest bidder of an auction. This is bad for gas, and the architecture here should be restructured into a mapping or a more efficient lookup algorithm.
## Style comments
Modifiers should be used instead of require statements in order to improve code readability. Natspec can be added to functions, and comments should be added to structs and state variables.
## Conclusion
The protocol should address the issues stated here such as limiting admin functionality, reducing the complexity of interactions and reducing the state-sharing between `MinterContract` and `NextGenCore`, and implementing reentrancy guards.









### Time spent:
16 hours
NextGen is a platform for launching generative art projects: collections of art with programmatically generated media, determined by a script written by an artist, and a randomizer that ensures dynamic and unique output for each piece.

Generative art has existed for decades, but has found a foothold in the NFT space in the last several years. Along with that popularity and value has come some big challenges: how can the creator of a popular generative art collection prevent bots from minting the whole supply? How can they make sure their work can be acquired at a reasonable price by their real community? How can they resist people using their work for short-term flips on secondary markets and losing track of the whole purpose of the project — the art — entirely?

These are the key problems that NextGen aims to solve, with a dynamic approach to minting processes and sales models that allows for flexibility and modularity in approaching these problems: artists can use allow lists, airdrops, "mint passes," and auctions to sell their work with either a fixed or a descending price, and configurable durations, total supplies, and allowances per account.

Most of the functionality is split between two contracts: `NextGenCore`, the foundational layer of the system and the token contract itself, and `NextGenMinterContract`, which acts as both the user interface for `Core` and the accounting and payment system for artists and the NextGen team. Additionally, the majority of these functions are restricted to role-based admins, defined in `NextGenAdmins` — an `Ownable` but otherwise bespoke approach to user authorization.

Each token gets a randomized hash when it mints to guarantee unique output from the generative script. This hash gets its randomness in one of three ways, defined by the collection owner on a per-collection basis: Chainlink VRF, ARRNG, or a NextGen randomizer that does not require an external call. Each randomizer has its own NextGen-authored contract for interfacing with its source of randomness, one of which is designated in the `collectionData` for each collection.

The last contract in scope is the `auctionDemo`, where all the logic for operating an auction (after `NextGenMinter.mintAndAuction` has created it in the first place) is implemented.

Test coverage is limited to minimal Hardhat unit tests. Natspec documentation is also sparse.

**Observations and recommendations**

*Function complexity*

Many of the key functions in these contracts are quite long, and contain several otherwise independent responsibilities — for example, `NextGenMinterContract.mint` handles everything from checking the new token's validity all the way to regulating sales methods explicitly.

While refactoring such things might seem like a non-essential task with a tight timeline before launch, the opposite is actually true. In fact, most of the major vulnerabilities in this codebase can be attributed, in one way or another, to the monolithic quality of these functions. They make it easy to execute logic in the wrong order, they make it hard for developers to track everything that happens in the execution of the function, and, looking forward, they will increase the likelihood that vulnerabilities remain hidden in the code even after several rounds of review.

Extracting logical tangents into their own functions will also have the added benefit of making the system more easily extensible and more modular, both of which seem to be goals of the NextGen project. Implementing a function that handles all the logic related to periodic sales, for example, would instantly simplify `Minter.mint` as well as the sales option itself, and it could facilitate new minting mechanisms as NextGen continues to experiment.

*Architecture and storage*

Likewise, the bounds of what should be defined and stored in the `Minter` or the `Core` are not clear. The result is similar, and best exemplified by the abundance of mappings and structs in both contracts.

Collection data is split between the two contracts ad hoc, so reviewing them means constantly guessing where some data is stored, often getting it wrong, and trying again. Worse, it obfuscates the relationship between each of these items in state, further complicating the process of defining and testing meaningful invariants in the code.

Consider extracting some functionality from both contracts: the storage, for instance, can be in a contract of its own and shared by `Minter` and `Core`. Many of these mappings can also be packed together in structs, and with so many instances throughout, it would make an appreciable difference in the cost of deploying the contracts (at the very least). It would also make sense to have a contract dedicated solely to the implementation of the sales options and price calculation. These are only examples — many similar refactoring opportunities exist throughout the codebase.

*Admins*

The `NextGenAdmins` contract mostly succeeds at what it aims to do, but since there are no special requirements for the contracts' authorization mechanism, it is not clear why it actually needs to have been built by NextGen in the first place. Using OpenZeppelin's `AccessControl` instead of `Ownable` with `NextGenAdmins` would come with all the functionality needed here, plus the added benefit of having already gone through plenty of careful review.

There is also a high degree of centralization risk, since the means exist both for the global admin and a particular function admin (discussed in a vulnerability) to withdraw all funds from `NextGenMinterContract` without any safeguards.

The team has stated that they intend to use multisigs for all admin accounts, which is a great idea. But alone, it is insufficient to eliminate the centralization risk in this code, since it ultimately still requires trust in a central authority to follow these informal standards.

There are undeniably practical considerations that justify admin privileges throughout these contracts. But some high-risk functionality warrants more hard-coded security than the standard approach from the rest of the codebase, even if the intended global admin of NextGen is the most trustworthy person in the world. There will be forks, and the team will change, and the honeypot in the contract might explode in value. And even if every person involved until the end of time *does* turn out to be trustworthy, showing NextGen users that the platform is secure by improving just a few of the most high-risk areas will not have been a waste.

Incidentally, there is already a more careful, multi-party process applied to proposing the addresses and royalty percentages associated with each collection. Simply applying a similar mechanism, with a requirement for approval from multiple admins, where funds can be withdrawn (and where the admin contract can be updated) would be a huge improvement on its own.

*Implicit assumptions and unexpected usage*

In addition to the assumption that admins are trusted and will always use multisigs, there are other assumptions the team has mentioned that are not supported by the code. Another example of this is the expectation that the phases of a mint will not overlap, as communicated by the team in Discord but not enforced in the contracts.

While this would be important to be aware of in any context, it is an especially serious consideration for NextGen, because they are actually launching with *two* instances of these contracts. The second is a fork operating an entirely separate product — certificates in the form of NFTs from a project called UNIC. Unless UNIC and NextGen have the same team behind the curtain, this compounds issues like unofficial security requirements, assumptions about user behavior, and faith that admins will always know how to use the system properly: now there is no reason to assume every admin was involved in the development of these contracts, or that they inherit any trust or security from the NextGen team.

It is impossible for someone outside the NextGen team to do a full accounting of such assumptions made internally, but doing so is a worthwhile exercise for the team. Assumptions like these are blind spots that can often be exploited.

*Miscellaneous recommendations*

In addition to these larger items, there are a number of areas where the NextGen code could be augmented to boost security, improve user experience, and prevent unexpected errors and reversions.

- Tests: Full unit test coverage is the minimum, but the team should also consider adding fuzz tests and invariant tests.
- Natspec documentation: Every function and every parameter should be documented in the code.
- Events: every major state update should emit events, and these events should be monitored by the team in perpetuity.
- Errors: when a user's action does not match the behavior the contract is expecting, the contract should revert with custom errors. Additionally, the existing error messages should be rewritten to provide greater clarity — "Change no of tokens," for example, does not tell the user what went wrong with the number of tokens they provided.
- Parameters: it is often better to have two versions of a similar function than to require null values from the user. For example, consider having a separate function for minting with a delegate, so non-delegated users are not required to input an empty address as a parameter in the majority of cases when this parameter is irrelevant.

Implementing these things will enable the next review to focus more on subtle, application-specific logic, and it will increase the odds that most major exploits are actually found and patched before launch.

**Takeaways**

Aside from the perennial value of tests and documentation, one interesting insight about `block.timestamp` also emerged. While common knowledge says `block.timestamp` is a less risky value now that it is not susceptible to miner manipulation, in situations where timing an attack is important, `block.timestamp` is actually more dangerous to rely on now that it can be calculated with certainty in advance. In NextGen, this can be exploited to cancel a winning bid in an auction after claiming the auctioned NFT.


### Time spent:
40 hours
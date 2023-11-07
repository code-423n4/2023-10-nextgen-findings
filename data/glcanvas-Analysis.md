# Codebase quality analysis

The overall quality of the codebase for NextGen can be classified as "Need work".

* At the first want to notice curious contracts usage. All contracts and interfaces placed in
  single folder without any division.
  Current approach significantly decrease readability. Better split it into
    * Interface directory;
    * Random directory;
    * Auction directory;
    * Core directory.

* Next item to notice is code formatting. Devs should follow widely used code style. To resolve this
  problem either use IntelliJ IDEA formatting
  or [npm plugin](https://github.com/prettier-solidity/prettier-plugin-solidity).

* Another one issue is copy-paste OpenZeppelin Contracts, for this purpose better to use Hardhat or
  Foundry.

* Last one is naming. Use UpperCamelCase for contracts and lowerCamelCase for modifiers. Moreover,
  contract's name should be the same as file name.

* And the last one current contracts implementation doesn't have corresponding interfaces for easily
  code navigation.

# Approach taken in evaluating the codebase

To evaluate an audit code base was used:

* Manual code review;
* Foundry for proof of concept;
* Remix IDE to assumption checking.

# Architecture recommendations

At the first want to note `NextGenAdmins` contract, from my point of view current idea of authority
is one of the best, because:

* You can observe all accesses in one place;
* Improves readability it is easy to audit all changes;
* This approach helps to easily add or remove rights for user in all contracts.

Another contracts divided correctly. Each of them does only what it needed to do, and nothing more.

But idea to mint next element based on MinIndex and
Supply `collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);` is
bad, because such approach adds more cases to make mistake during copy-paste. Better will be adding
view function to Core contract, which returns next token to mint
like `function nextTokenToMint(uint256 _collectionId)`, and reverts if collection is full.

Next bad pattern is Core contract during mint relays in Mint contract. In current implementation all
is correct, and corner cases handled normally, but in future version from minter contract some
checks might be removed occasionally this leads to overflow variable `collectionCirculationSupply`.

# Centralization risks

Current project is significantly centralized. This means that a lot of function can be called only
by owner or trusted admins.
This is not a good way to resolve possible issues. And, for example, collections can be created by
anyone or artist.
Moreover, collections details might be filled by artist too. Because in core users might create up
to 10_000_000_000 collections.

Let's sum up:

* Now only admins can do all stuff (create collection, add info, and so on);
* This responsibility might be passed to collection owners too;
* Now only admins can create collections, better allow to create collections by anyone.

# Mechanism review

NextGen is system of contracts, which can create and hande multiple ERC721 collections.

Core contract handles multiple ERC721 collections in single contract.
This achieved by mapping `collectionId` on single line of `tokenIds`. Each collection has own
continuous interval.
This restricts amount of collections up to 10_000_000_000, this is more than enough.

It gives possibility to:

* Do all ERC721 stuff (burn tokens, mint tokens, transfer tokens);
* Create collections;
* Set randomizers for hash calculation;
* Set collection metadata;
* Set artists.

Moreover, during minting users can calculate their token hash based on random.
These handles during calling one of randomizer. Current workflow is:

* User mints token;
* User calling random;
* Random calling underlying system Arrng or ChainLink;
* Callback filling hash in Core contract.

----


Then, another cornerstone is Minter contract. This contract linked with Core contract and processes
minting, burning tokens for corresponding collections.

Minter contract has different possibility to mint, via:

* Mint when allow list market, to buy NFT user needs to provide merkle proof with maximum possible
  tokens to buy;
* Mint when public market;
* Mint when burning, this gives possibility to exchange one NFT to another one, this handles during
  burning old one and minting new one. Burned tokens can either one of NextGen collections or any
  external collections.

Next, minter gives possibility to distribute reward to artist and team.

To buy NFT users need to send a bit of ether to contract. Price estimation lies on this contract
too.

Price can be either increase `salesOption = 3` or decrease  `salesOption = 2 & allowList stage` or
be constant `collectionMintCost`.

* Price increase depends on current supply. THe more tokens in circulation, the higher the price. So
  this function is linear: `price = offset + r * X`, where `X` is `circulation`;
* Price decrease depends on time and is exponentially function. Like: `price = initialPrice / time`.

# Systemic risks

* Unavailability of Arrng or ChainLink. This may stop filling token's hash. And current
  implementation doesn't have any late filling mechanism.
* Data inconsistency. Current NextGen implementation gives possibility to replace any system, like
  change Minter in Core contract and vice vera.
* Like any smart contract-based system, NextGen is exposed to potential coding bugs or
  vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the
  protocol.
* Big centralization. All significant functions can be called only by admin team, this does not
  correspond to the idea of decentralization that web3 is trying to create.
* Lack of any input verification. Due to all create/modification functions under admin modifier, any
  verification absent. This likely to have fat finger problem.

### Time spent:
30 hours
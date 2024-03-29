## NextGen overview

The NextGen protocol allows for the creation and distribution of generative art NFTs according to different mechanisms. All NFT collections reside in the same ERC-721-derived contract, `NextGenCore`. Minting, airdrops and handling of royalties is done by a separate contract, `NextGenMinterContract`, and the protocol's administration is controlled by `NextGenAdmins`. These three contracts form the core of the protocol.

Periphery contracts include randomizers, which provide randomness for art generation using different methods, and can be assigned on a per-collection basis; an auction contract, used to auction tokens to the highest bidder; and `DelegationManagementContract`, which allows for delegation by token owners to other parties. This final contract was out of scope for the Code4rena contest.

## Centralization risk

Operation of the NextGen protocol is highly dependent on administrators. The admin contract owner and appointed global administrators have the ability to execute any of the protocol's functions and make almost any change to the protocol at any time. The degree of power held by administrators and the protocol's reliance on them negates many of the inherent strengths of Ethereum's decentralization.

While it is common for protocols to have administrative functions and components that require active upkeep, more decentralized protocols empower users to do as much as possible, and the most decentralized protocols are designed to continue functioning even in the event that they are abandoned by developers. This is not the case for NextGen -- artists cannot create or configure their own collections, as all the functionality for this is available only to protocol administrators. This reduces artist control over their collections and ensures that the protocol will be unusable if abandoned by its developers.

Overall, rethinking the permission structure and workflows involved in creating a collection could increase the protocol's longevity and reduce the workload on NextGen administrators. Even if the protocol admins behave entirely ethically and diligently maintain the protocol for its entire lifespan, there remains a risk of compromise. The compromise of an administrator account could have devastating effects on the protocol.

The lack of events for the majority of functionality could also contribute to a perception that the protocol is not transparently run.

Below, some commentary on individual functions:

### `NextGenMinterContract.emergencyWithdraw`

This function allows a protocol admin to withdraw the entire Ether balance of the minter contract to the owner's address at any time. The `Withdraw` event it emits may be considered misleading, as it includes only the address which called the function, which may not be the address which received the funds.

Even if we assume trust on behalf of the admin addresses empowered to call this function, its presence could deter artists and other users from engaging with the protocol due to its obvious potential for abuse.

Users' and artists' potential concerns with this functionality could be mitigated by including conditions on its successful execution, such as a timelock of at least one year.

### `NextGenMinterContract.payArtist`

This function must be called by a protocol admin to pay artists and teams their share of the Ether generated by a given collection. The active function of the protocol requires administrators to call this function periodically for every active collection. This allows administrators a large amount of power over when artists and teams are paid, and also prevents self-service use of the protocol by artists. It also opens up the possibility for arbitrarily different team addresses and percentages to be specified on each call.

A more conventional pattern for this process would be to implement an `claim` function which artists and teams could call to claim their shared of funds for their collection. This could be implemented in such a way that administrators are still able to call it to pay the artist of a given collection, thus preserving current functionality. This would require team addresses and percentages to be stored in the contract beforehand.

### `NextGenCore.addMinterContract`, `NextGenCore.updateAdminContract`, `NextGenMinterContract.updateCoreContract`, `NextGenMinterContract.updateAdminContract`

These functions allow protocol admins to change individual contracts used by the protocol. This can be done at any time, with the only requirement being that the new contracts implement a single function that returns `true`. Administrators could use this functionality to unilaterally and maliciously change the behavior of the protocol. The possibility of this eventuality reduces user trust, even if no malicious actions are ever taken by the protocol admins.

### `NextGenMinter.setCollectionCosts`, `NextGenMinter.setCollectionPhases`

Both of these functions can be called repeatedly with different values by either the relevant collection or function admins, or the global admins. This would allow mint costs, sales options and collection phases to change, even during an active minting phase.

Per the documentation, the protocol's intention is to allow an arbitrary number of allowlisted and public phases with different sales models. To allay user concerns about abuse of this functionality, conditions could be added to prevent these functions from being called during an active allowlist or public sales phase.

### `NextGenMinter.proposePrimaryAddressesAndPercentages`, `NextGenMinter.proposeSecondaryAddressesAndPercentages`

These functions can be called by the artist of a given collection, or by a function admin or the global admin for any collection. Allowing them to be called only by the artist would give artists greater assurance in their ability to control their payments.

## Controls and patterns implemented

This section provides commentary on notable controls and patterns implemented in the NextGen codebase.

### Function admin pattern

NextGen makes extensive use of an administration pattern where the global administrator can assign rights to additional administrators on a per-function basis. This allows for granular access management, but has a couple of potential pitfalls:

1. `registerFunctionAdmin` assigns function admins by storing an association between a function selector and an address. This is used for access control across multiple contracts. Therefore, if two or more contracts in the protocol contained functions with the same name and parameters, a function admin would be authorized to execute all of them, as they would have the same selector. This is already the case for `NextGenMinterContract.emergencyWithdraw` and `NextGenRandomizerRNG.emergencyWithdraw`, and should be kept in mind when assigning that permission, as well as during any further protocol development.
2. While assigning an administrator control of only a single function narrows the scope of how they can interact with the protocol, many individual functions are quite powerful. A function administrator with access to `setCollectionPhases`, for example, can call this function for any collection. Furthermore, function administrators with access to protocol-changing functions such as `NextGenMinter.updateAdminContract` or `NextGenCore.addMinterContract` should be considered to have privileges essentially equivalent to global administrators.

### Randomness

NextGen provides three different randomizer contracts:

* `NextGenRandomizerVRF`: Provides random words via the Chainlink VRF oracle
* `NextGenRandomizerRNG`: Provides random words using the ARRNG oracle
* `NextGenRandomizerNXT`: Provides random words using on-chain data

Global and function administrators can assign any of these randomizers to any existing collection, and can change a given collection's randomizer at any time.

The documentation treats all three randomizers as essentially equivalent, but the `NextGenRandomizerNXT`'s behavior diverges significantly from the other two. Rather than receiving random values from an oracle, it computes randomness on-chain, in the two functions below:

```solidity
    function randomNumber() public view returns (uint256){
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
        return randomNum;
    }

    function randomWord() public view returns (string memory) {
        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
        return getWord(randomNum);
    }
```

The values used for the randomization, `block.prevrandao`, the `blockhash` of a previous block, and `block.timestamp` are all either known or controllable to some extent by miners and will lead to a truly random output.

This issue is mitigated by the fact that randomness has only a cosmetic function in the protocol and there are no direct financial incentives for manipulating it. However, clearly documenting the relative strengths and weaknesses of each of the generation methods would help artists to make a more informed decision about which one to use for their collection.

### isContract checks

Many of the in-scope contracts implement a method named `isNAMEContract` (e.g. `isMinterContract` and `isRandomizerContract`). This method returns `true` and is called when adding or updating contract addresses. The method's presence is not a foolproof way of ensuring that the contract is of the right type, as a malicious contract could implement the same method, or return `true` in its fallback method. As contract changes should be fairly rare occurences, aside from setting randomizers for collections, the majority of these methods could be safely removed. Any remaining functions of this nature could be converted to public boolean constant state variables set to `true`, to save on gas costs and contract size.

If this functionality is strictly required, it could be implemented more robustly by supporting [ERC-165](https://eips.ethereum.org/EIPS/eip-165).

### maxCollectionPurchases

Collections have a `maxCollectionPurchases` value which limits the number of tokens a given address can mint. This has two potential flaws:

1. Allowlisted users who have already minted tokens during phase 1 of the sale would be able to mint additional tokens during the phase 2, without their existing token holdings being taken into account. It is acknowledged that this may be intended functionality.
2. A single user could purchase additional tokens beyond `maxCollectionPurchases` by using more than one EOA.

If the purpose of this control is to limit the number of tokens a single user can mint, it is ineffective. If the purpose of this control is merely to make it more difficult to mint all available tokens in a single transaction, it should serve this purpose. Furthermore, this second purpose could be served by the below line in `NextGenMinterContract` on its own, with the `tokensMintedPublicPerAddress` accounting being unnecessary:

```solidity
require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
```

### Collection generation script

Function and collection admins could provide a collection script for collections. This script is intended to generate the token's artwork, but could contain malicious functionality. The function `retrieveGenerativeScript` outputs the collection script as part of a JavaScript string, which may be run in users' browsers. Care should thus be taken to ensure that collection scripts do not contain malicious JavaScript, such as keyloggers or code that attempts to interact with user wallets.

## Architectural recommendations

This section provides a few high-level recommendations for streamlining NextGen's architecture and improving code quality.

### High-level refactoring suggestions

Per the bot report and my own QA report, the following high-level suggestions could be followed to reduce code size and complexity:

1. Replace simple getter functions with public variables, as Solidity automatically creates getter functions for these.
2. Avoid storing both a contract's address and interface in separate variables. Rather cast the interface to an address where needed.
3. Where possible, check for the existence of a struct or mapping entry by checking a value that must be non-zero instead of including a boolean variable solely for this purpose.

### Function complexity

NextGen allows a variety of different methods for minting tokens, spread across the following functions in `NextGenMinterContract`:

* `airdropTokens`
* `mint`
* `burnToMint`
* `mintAndAuction`
* `burnOrSwapExternalToMint`

And the following functions in `NextGenCore`:

* `airdropTokens`
* `mint`
* `burnToMint`

These functions all contain significant amounts of duplicated code and could be refactored using multiple private or internal functions to reduce this code duplication.

Complexity could also be reduced by introducing separate functions for allowlist and public phase minting. This would remove the need for public phase minters to provide unused function arguments such as `_maxAllowance` and `merkleProof`.

Similarly, the function `NextGenCore.updateCollectionInfo` could be refactored into multiple functions instead of using an `_index` parameter. This would increase the readability of the codebase and the legibility of the contract's external API, as represented by its public and external functions.

### AuctionDemo

The AuctionDemo contract could be greatly simplified, though at the expense of some functionality. Instead of storing all bids and allowing multiple bids per address, the function `participateToAuction` could compare the proposed bid with a stored highest bid value (starting at 0). If the proposed bid is lower, the function reverts. If the proposed bid is higher, it becomes the new highest bid, and the caller becomes the new highest bidder. Once the auction finishes, the token is distributed to the highest bidder and the payment to the minter.

While bidders would not be able to cancel their bids in this version of the contract, it would remove the requirement for complex logic in the contract's view functions, as the highest bid and bidder could be stored in a public mapping. It would also remove the requirement for complex logic in the `claimAuction` function, for the same reasons.

In this way, the contract size could be reduced by approximately 50%.




### Time spent:
20 hours
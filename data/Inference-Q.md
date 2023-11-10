# Unclear spec re MintedALPerAddress and MintedPublicPerAddress 

## Smart contract
NextGenCore.sol and MinterContract.sol

## Rating
Unknown, since depending on particular intentions/specification.

## Observation
Based on the available documentation, the precise purpose of "MintedALPerAddress" and "MintedPublicPerAddress" is not clearly defined. It remains particularly unclear whether these checks have been exclusively implemented within the "mint" function of the "MinterContract" and not in other functions such as "burnToMint" or "burnOrSwapExternalToMint."

## Proof of concept / attack scenario  
None.

## Tools used
None.

## Recommended mitigation steps
Verify whether the implementation aligns with the intended purpose, and add the necessary checks regarding "MintedALPerAddress" and "MintedPublicPerAddress" if deemed necessary.

# Random number generation
## Smart contracts
RandomizerNXT.sol and XRandoms.sol

## Rating
Unknown, since depending on use / business case, which is unknown to us.

## Observation
The validator responsible for the current block has the capacity to manipulate the block's timestamp. Consequently, a validator could intentionally select the timestamp to influence the generation of the random number.

## Proof of concept / attack scenario  
Validators may be inclined to manipulate the random number to obtain a value associated with a rare NFT.
 
## Tools used
None.

## Recommended mitigation steps
It is advisable to utilize this random number generator solution only for scenarios where the random number is not linked to rarity or when the potential cost of executing such an attack is prohibitively high compared to the potential returns.

# Items have to be auctioned by AuctionDemo contract
## Smart contract
AuctionDemo.sol & MinterContract.sol

## Rating
Unknown, since depending on use / business case, which is unknown to us.

## Observation
The "mintAndAuction" function in the MinterContract accepts a parameter named "_recipient," which designates the recipient of the airdropped token to be auctioned. It is essential that this parameter is always set to the contract owner of the "auctionDemo" contract. Failure to do so would violate the following key premise outlined in the scope description:

**"The highest bidder will receive the token after an auction finishes, the owner of the token will receive the funds, and all other participants will be refunded."**

This is because, in the "claimAuction" function of the AuctionDemo contract, the funds are disbursed using the following code:

`(bool success, ) = payable(owner()).call{value: highestBid}("");`

It is critical that the auctionDemo contract is always the owner of the NFT, thus the “_recipient” in the function ”mintAndAuction”. If this is not ensured this would lead to the issue that the AuctionDemo contract is not able to transfer the auctioned NFT to the highest bidder due to missing permissions, but potentially also other security concerns (like e.g. reentrancy attacks), since we have not assessed the security with regards to such a setup.

It is of paramount importance to ensure that the AuctionDemo contract maintains initially the ownership of the airdropped NFT, as reflected in the "_recipient" parameter of the "mintAndAuction" function. Failure to adhere to this requirement could result in significant issues. For instance, it may lead to a situation where the AuctionDemo contract lacks the necessary permissions to transfer the auctioned NFT to the highest bidder and also potentially posing other security concerns (like e.g. reentrancy attacks). We have not evaluated the security implications of this specific configuration.

Therefore, our understanding is that the "FunctionAdmin" will consistently designate the AuctionDemo contract as the "_recipient" when invoking the "mintAndAuction" function in MinterContract.sol. However, this configuration poses the risk that the NFT may become trapped within the AuctionDemo contract in situations where there are no bids or all bids are canceled.

## Proof of concept / attack scenario  
None.

## Tools used
None.

## Recommended mitigation steps
Evaluate the situation to determine if the current code aligns with your specifications.

If it's confirmed that "_recipient" is consistently the "AuctionDemo" contract, you should consider one of the following options:
1) Incorporate a check in the smart contract code to validate that "_recipient" is indeed the "AuctionDemo" contract.
2) Alternatively, document the potential issue and consistently educate the "FunctionAdmin" to refrain from specifying any address other than the "AuctionDemo" contract.

Furthermore, contemplate the implementation of a recovery process for NFTs that haven't received any valid bids, if it is deemed worthwhile to recover such NFTs.

If the intention is to allow "any address" as the "_recipient," we strongly recommend reviewing this specific code, implementing changes, and evaluating the potential security risks associated with this setup.

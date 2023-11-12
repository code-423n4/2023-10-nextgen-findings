# [01] - In `AuctionDemo` contract, a malicious user can carry out a block-stuffing attack to ensure that he is the highest bidder.
A malicious user can carry out a block-stuffing attack to ensure that he is the highest bidder. For example a very rare/expensive Token/NFT is auctionned. The highest bid is more than 50ETH and our attacker Bob absolutetly want this token. He check is nearly the end of the auction time, he place a higher bid of 55ETH and become the new highest bidder.
Now he can check the mempool and make a blockstuffing attack by sending dummy transactions with a lot of gas to prevent the potential `participateToAuction()` transactions to be integrated into the chain before the end of the auction.
I consider it at low because a low likelihood and because gas cost is expensive on the Ethereum blockchain.

More infos here: https://consensys.github.io/smart-contract-best-practices/attacks/denial-of-service/#gas-limit-dos-on-the-network-via-block-stuffing

# [02] - In `MinterContract::setCollectionPhases()`, lack of verifications can lead to potential issues on the minting process
Lacks of verification inside the `MinterContract::setCollectionPhases()` functions can lead to potentials time issue on the minting process.
Examples if times parameters are misconfigured are impossibility for user to mint, computation problems inside the `getPrice()` function, etc.

## Recommendations
Add verification like `_allowlistStartTime < _allowlistEndTime` and `_publicStartTime < _publicEndTime`.
You can verify also the periods are greater than `block.timestamp`.

# [03] - Function `MinterContract::airDropTokens()` doesn't check for inputs arrays have the sames sizes
The function `MinterContract::airDropTokens()` is a function that allows an authorized admin to execute multiples minting in a airdrop for multiples users. It requires multiples inputs and some of them are dynamic sized arrays, and because size of some arrays are not checked to be at the same length, it can lead to minting with `_tokenData[]` or `_saltfun_o[]` with zero values, or no minting if the `_numberOfTokens[]` is less than others arrays (and equal 0).

## Recommendations
Add verification to check if the `_recipients[]`, `_tokenData[]`, `_saltfun_o[]` and `_numberOfTokens[]` have the same length.

# [04] - Rounding error lead to bad distribution of royalties and losing ETH

`MinterContract::payArtist()` is the function called by admin to pay the artist and the team percentage in sending the deposited amount for a collection. We look that the function makes the following operations to divided the total royalties to the artist and the team.

```solidity
    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        ...
        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
        teamRoyalties1 = royalties * _teamperc1 / 100;
        teamRoyalties2 = royalties * _teamperc2 / 100;
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
        ...
    }
```

Take this scenario with this percentage separation and this little amount: 

>   Artist: 60% 18% 2%
    Team: 10% 10%
    royalties = 100495

    artistRoyalties1 = 100495 * 61 / 100
    artistRoyalties2 = 100495 * 17 / 100
    artistRoyalties3 = 100495 * 2 / 100
    teamRoyalties1 = 100495 * 11 / 100
    teamRoyalties2 = 100495 * 9 / 100

    artistRoyalties1 = 61301
    artistRoyalties2 = 17084
    artistRoyalties3 = 2009
    teamRoyalties1 = 11054
    teamRoyalties2 = 9044

    Total) 61301 + 17084 + 2009 + 11054 + 9044 = 100492

We can see a difference of 3 wei, it not a big amount and it's for this reason I think this issue is low. 
But with a lot of call for multiples payment during times and calls for all the collections on the contract. Artist are impacted and lose amount of tokens while the dust ETH are losed in the contract.

## Recommendations 
Change the methods of computations and/or verify is the royalties amount is equal to all the computed amount and add the difference to one amount to prevent dust ETH in the contract. 

# [05] - In `RandomizerNXT` contract, the `updateAdminsContract()` don't verify if the contract is an admin contract
To follow the logic to check that a contract is an admin contract/minter contract/etc. In the `updateAdminsContract()`, when an admin change the admin contract, there is no check if the contract is an admin contract. By mistake, admin can change the admin contract to another address and the check with the function is not made.

## Recommendations

Add this line on the function :
```solidity
    function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {
        ++ require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_admin);
    }
```

# [06] - An attempt to mint with a `RandomizerRNG` contract can always revert

The `RandomizerRNG` contract not set in the constructor the minimum ETH required to make a call to the `ArrngController` (see below), you can set it directly in the constructor with the minimum value defined by the `ArrngController` controller in the `minimumNativeToken` (see the concerned parts of the Arrng controller below). If an admin forgot to set the `ethRequired`, all the users call will revert.

https://github.com/arrng/arrng-contracts/blob/117d2eadc55eaabb5bea58396dbc8f421d735fd5/contracts/controller/ArrngController.sol#L427
https://github.com/arrng/arrng-contracts/blob/117d2eadc55eaabb5bea58396dbc8f421d735fd5/contracts/controller/ArrngController.sol#L31-L35

## Recommendations

Set the `ethRequired` in the constructor of the contract to the minimal value to avoid the calls revert when a user try to mint.
At the time of writing `minimumNativeToken = 1000000000000000`.

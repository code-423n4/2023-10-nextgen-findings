### [L] Function `returnHighestBidder` returns the last element of auctionInfoStru[], not the highest bidder

The function `returnHighestBidder` compares the current bid in the loop not with the highest bid, but with 0, making the process of comparison pointless. However current implementation of the auction doesn't allow the following bidder to make a bid less than the previous bidder. As a result, the last member of the array with true status is always the highest bidder.

It works now, but if the logic of the auction is changed the last participant will become a winner.
In the correct implementation the highBid variable should be rewritten in case `auctionInfoData[_tokenid][i].bid > highBid.`

     function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) { //@audit doesn't rewrite
            
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
              +  highBid = auctionInfoData[_tokenid][i].bid; 
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }

### [L] Auction demo contract doesn't have functions that allow updating of minter, gencore and adminsContract addresses

The mentioned above addresses are set in the constructor, but there is no way for admins to update them, meanwhile, other contracts have this functionality.

### [L] It's not possible to cancel auction for the owner of the token.

The token that was minted for the auction is still owned by the user, so if he decides to cancel the auction he can remove approval for the auction contract. In this case, the auction will stay active and other users will be able to participate in it and send eth as bids. 

But at the end of the auction, the winner will not be able to claim his prize and other participants will be forced to withdraw their funds manually. This leads to frustration of the users and waste of gas.

### [L] `_saltfun_o` variable in the mint function should be different for every token:

The mint function performs the minting of tokens in an array using the same `_saltfun_o` variable for every token:

     for(uint256 i = 0; i < _numberOfTokens; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
            
            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase); //@audit salt fun should be different for every token
        }

In the current implementation, this variable is not used. But in a similar function airDropTokens, we see that there is an array of salt.

    function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector)

### [L] Function setCollectionCosts from the minter contract doesn't check if collectionMintCost is more than the rate.

The function `getPrice` for the phase 3 has the following logic:

     function getPrice(uint256 _collectionId) public view returns (uint256) {
        uint tDiff;
        if (collectionPhases[_collectionId].salesOption == 3) {
            // increase minting price by mintcost / collectionPhases[_collectionId].rate every mint (1mint/period)
            // to get the price rate needs to be set
            if (collectionPhases[_collectionId].rate > 0) { //@audit collectionMintCost should be <rate
                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
            } else {
                return collectionPhases[_collectionId].collectionMintCost;
            }

It should "increase minting price by mintcost / collectionPhases[_collectionId].rate".
So If `collectionPhases[_collectionId].collectionMintCost < collectionPhases[_collectionId].rate` the result will be equal zero, i.e. the price won't increase. 
I report this as Low because it's hard to imagine that the mint cost will be less than the rate.


### [L] Function setCollectionData from the NextGenCore should check if the `_maxCollectionPurchases` is less than `_collectionTotalSupply`.

By using the variable _maxCollectionPurchases admin can modify the amount of the tokens that could be purchased from a collection by a user. But this number can't be more than the total supply.

The only place in the code where this variable is used is in the mint function:

    226: require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");

The idea is that the allowed amount per user can be changed at any moment, but the setCollectionData function allows admins to set this parameter to be greater than the total supply making this require statements pointless.

### [L] Require statements in addMinterContract and updateAdminContract functions are too simplistic, making them susceptible to circumvention.
 
Before setting new minter and admin addresses the mentioned above functions make some checks in the require statement:

     function addMinterContract(address _minterContract) public FunctionAdminRequired(this.addMinterContract.selector) { 
        require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter"); //@audit always true
        minterContract = _minterContract;
    }

    // function to update admin contract

    function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin"); //@audit always true
        adminsContract = INextGenAdmins(_newadminsContract); 
    }


If we take a look at these `isMinterContract` and `isAdminContract` functions we see the following:

    function isMinterContract() external view returns (bool) {
        return true;
    }

    function isAdminContract() external view returns (bool) {
        return true;
    }

They always return true without checking any data. These checks provide a false sense of security and lack practical significance

### [L] Using 100 as a divisor in the function payArtist() can lead to the wrong amounts being paid. Also, some additional checks to prevent 0 payments should be added.

In Solidity, using 10000 as a divisor when calculating percentages is a common practice to avoid precision issues with integer arithmetic. Solidity primarily deals with integers, and using 100 as a divisor might result in rounding errors due to limited precision. By using 10000, developers can achieve greater precision in their calculations.

This function 5 times calculates percentages from the same value. Several cases of rounding down will lead to the dust left in the contract. To avoid this issue use 10000 as a divisor and 10000 as 100%.

Also, there are 5 division operations in the payArtist() function. In some cases it can lead to 0 payments, to prevent this the numerator should be more or equal to 100:

     artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
     artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
     artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
     teamRoyalties1 = royalties * _teamperc1 / 100;
     teamRoyalties2 = royalties * _teamperc2 / 100;


### [QA] It is not possible to retrieve the token balance from a specific collection 

The protocol's functionality involves various collections, each associated with a specific list of tokens. However, during testing, I observed that there is no way to monitor a user's balance in different collections. Since all collections can significantly differ from one another, I believe it would be convenient for users to see not only the total token count but also the token balance from each collection.
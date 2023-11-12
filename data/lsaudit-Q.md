# [QA-01] The name of `addRandomizer()` function from `NextGenCore.sol` may be misleading

[File: NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L170-L174)
```
    function addRandomizer(uint256 _collectionID, address _randomizerContract) public FunctionAdminRequired(this.addRandomizer.selector) {
        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");
        collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
        collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract);
    }
```

As demonstrated above, `addRandomizer()` does not only add new randomizer - but when randomizer is already added - it will be updated.
Our recommendation is to change the name from `addRandomizer` to `addOrUpdateRandomizer()`.


# [QA-02] Incorrect comment for `setCollectionData()` function in `NextGenCore.sol`

According to comment: 

[File: NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L145)
```
// only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set
```

However, the current implementation allows to change not only `_collectionArtistAddress` and `_maxCollectionPurchases`, but also `_setFinalSupplyTimeAfterMint`

[File: NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L168-L165)
```
 } else if (artistSigned[_collectionID] == false) {
            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
        } else {
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
```


# [QA-03] It's not possible to unfreeze previously freezed collection

`NextGenCore.sol` implements only one function which allows to freeze a collection. However, there's no additional function which would allow to unfreeze it.
This may lead to some issues when collection is frozen by mistake.

[File: NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292)
```
 function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col");
        collectionFreeze[_collectionID] = true;
    }
```


# [QA-04] Function `emergencyWithdraw()` has hardcoded address which cannot be changed

[File: RandomizerRNG.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L79-L84)
```
function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }

```

`emergencyWithdraw()` allows to withdraw any balance from the smart contract. The address where funds will go are defined as `adminsContract.owner()`.
Since it's not possible to change the owner of `adminsContract`, this address is non-changeble. If private key will be stolen or lost for this address - it won't be possible to access funds withdrawn to this address. 
Re-implement `emergencyWithdraw()` so that it will be possible to set any address where funds might be withdrawn.

The same issue occurs in `MinterContract.sol`:

[File: MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L461)
```
 function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```


# [QA-05] `getWord()` from `XRandoms.sol` does not provide correct probability distribution

[File: XRandoms.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L28)
```
if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
```

`getWord(0)` and `getWord(1)` will return the same data. That implies, that the distribution is not the same for every `id`.
There's no need to perform `wordsList[id - 1]`. Returning `wordsList[id]` every time will be sufficient. Especially, that function `getWord()` is used in `randomWord()` function which performs `% 100` on `id`.
This means that `0 <= id < 100`, so we do not need to care about `getWord(100)`, since it's not possible to reach it.


# [QA-06] `setFinalSupply()` does not verify if collection exists


[File: NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L307)
```
   function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```
`setFinalSupply()` will execute on non-existing collections.

This does not pose any security threat - thus it's evaluated as Low. Nonetheless, our recommendation is to add additional check: `require((isCollectionCreated[_collectionID] == true), "Not allowed");`



# [QA-7] Lack of data validation in `setCollectionPhases()` in `MinterContract.sol`

[File: MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L172-1L)
```
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
```

Function `setCollectionPhases()`  is used to set the starting and ending times of both the allowlist and public phases.
However, there's no verificaiton of provided data:

* User may set `allowlistStartTime > allowlistEndTime`
* User may set `publicStartTime > publicEndTime`
* According to documentations, allowlist phase should be sooner than public phase, but user may set  `allowlistStartTime > publicStartTime`

This is user-related error (user calls function with incorrect parameters) - thus it was evaluated as Low.

# [QA-8] Incorrect data emitted in `Refund()` event in `AuctionDemo.sol`

[File: AuctionDemo.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L117)
```
emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
```

Function does not refund `highestBid`, but `auctionInfoData[_tokenid][i].bid`.
Above line of code should be changed to:

```
emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, auctionInfoData[_tokenid][i].bid);
```

# [N-01] Separated parameters by comma should have white-space

A good coding-style practice is to divide parameters separated by comma by extra white-space, e.g.: `A,B`, should be changed to `A, B`.

[File: MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L316)
```
bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
```

[File: MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L327)
```
bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
```

Change `_erc721Collection,_burnCollectionID` to: `_erc721Collection, _burnCollectionID`.


# [N-02] Incorrect white-spaces in `AuctionDemo.sol` and `MinterContract.sol` loops

`AuctionDemo.sol`:
```
69:     for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
90:     for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
110:    for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
136:    for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
```

Lines 69, 90, 110, 136: change `i=0;` to `i = 0;`
Lines 69, 90, 110: change `i< auctionInfoData[_tokenid].length;` to `i < auctionInfoData[_tokenid].length;`
Line 136: change `i<auctionInfoData[_tokenid].length;` to ` i <auctionInfoData[_tokenid].length;`
Line 110: change `i ++` to `i++`

`MinterContract.sol`
```
184:    for (uint256 y=0; y< _recipients.length; y++) {
187:    for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
234:    for(uint256 i = 0; i < _numberOfTokens; i++) {
```

Lines 184: change `y=0; y< _recipients.length;` to `y = 0; y < _recipients.length;`
Lines 187, 234: change `for(uint256 i = 0;` to `for (uint256 i = 0;`
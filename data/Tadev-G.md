## [G‑01] Either `address gencore` or `INextGenCore public gencoreContract` should be removed from RandomizerNXT, RandomizerRNG and RandomizerVRF contracts.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L22-L23

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerRNG.sol#L21-L22

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerVRF.sol#L35-L36

Declaring both the address and the contract instance will use 2 storage slots instead of one. The constructor will assign 2 values with no reason : 
```
gencore = _gencore;
gencoreContract = INextGenCore(_gencore);
```

Moreover, the `updateCoreContract(address)` function updates both values, while only one would be enough.

If you only keep the `address gencore` variable, just wrap it into a contract type when you need to call a function on it. While RandomizerNXT contract doesn't even use `gencoreContract` and creates this variable with no reason, RandomizerRNG and RandomizerVRF contracts only use it once in `fulfillRandomWords()` function. You could just rewrite the call to `setTokenHash()` function in Randomizer contracts as follows : 

```
INextGenCore(gencore).setTokenHash(...)
```


## [G‑02] Useless `tokenToRequest` mapping in RandomizerRNG and RandomizerVRF contracts should be removed

Both RandomizerRNG and RandomizerVRF contracts declare a mapping `mapping(uint256 => uint256) public tokenToRequest`, and it is only used to store `requestId` in the `requestRandomWords()` function. But this value is never used, and I don't see any utility in storing the requestId corresponding to a minted token in the Randomizer contract.

While the bot report suggests to merge these following 2 mappings using struct/nested mappings in both contracts (N33 and G04 issues): 

```
    mapping(uint256 => uint256) public tokenToRequest;
    mapping(uint256 => uint256) public tokenIdToCollection;
```
I propose to simply remove `tokenToRequest` variable, as it is unused and has no utility. This will save gas compared to creating a merged data structure without the need to do it.


## [G‑03] Useless variables are created in `mint()`, `burnOrSwapExternalToMint()` and `payArtist()` functions in NextGenMinterContract and should be removed.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L198C7-L198C7

At the beginning of the `mint()` function in NextGenMinterContract contract, a variable called `col` is declared and set : 
```
uint256 col = _collectionID;
```
While `_collectionID` variable could be used in the rest of the function, declaring `col` costs gas without any reason. This variable should be removed, and `_collectionID` should be used wherever `col` is currently used.

The same applies for `tokdata` variable, which is declared and assigned to `_tokendata` : 

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L201

This string variable `tokdata` should also be removed, and `_tokendata` should be used wherever `tokdata` is currently used.

All this also applies for `burnOrSwapExternalToMint()` function, with the same 2 variables.

For `payArtist()` function, 3 variables are created inside the function, but this is redondant and they should me removed : 
```
address tm1 = _team1;
address tm2 = _team2;
uint256 colId = _collectionID;
```


## [G‑04] 2 variables are created and assigned to the same value in `burnToMint()` and `mintAndAuction()` functions in NextGenMinterContract. One of them should be removed.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L264

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L267

Both `collectionTokenMintIndex` and `mintIndex` variables are assigned to `gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID)`. 

This is redondant. Hence, `mintIndex` should be removed, as there is an important check just after `collectionTokenMintIndex` declaration and assignment. Also, `mintIndex` should be replace where it used : 
```
gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
```
should be:
```
gencore.burnToMint(collectionTokenMintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
```
This is the case in both `burnToMint()` and `mintAndAuction()` functions.


## [G‑05] Useless if statement in `mint()` function and `burnToMint()` function in NextGenCore contract should be removed

`mint()` function and `burnToMint()` function of NextGenCore contract both use an if statement to decide whether or not it should proceed to `_mintprocessing()` and achieve minting process : 

```
     if (
            collectionAdditionalData[_collectionID].collectionTotalSupply
                >= collectionAdditionalData[_collectionID].collectionCirculationSupply
        ) {...}
```
This check is actually not necessary, as:
- `mint()` function can only be called by `mint()` or `burnOrSwapExternalToMint()` functions of the Minter contract
- `burnToMint()` function can only be called by `burnToMint()` function of the Minter contract. 

All theses functions already make sure the totalSupply is not reached. Hence, there is no situation in which `mint()` function or `burnToMint()` function in NextGenCore contract can be called without satisfying the condition of this if statement.
Moreover, if calling theses functions was possible while totalSupply is reached, this would mean that it could be possible to increase `collectionCirculationSupply` value beyond `collectionTotalSupply`, as the `_mintprocessing` function wouldn't be executed while `collectionCirculationSupply` would be incremented by one, without any revert.

I suggest to remove this if statement that will consume gas for no reason.


## [G‑06] Unnecessary assignment of `collectionArtistPrimaryAddresses[_collectionID].status` variable to `false` in `proposePrimaryAddressesAndPercentages()` and `proposeSecondaryAddressesAndPercentages()` functions in NextGenMinterContract

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L389

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L403

Both functions use this check :
```
        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
```

Nevertheless, `status` is already false during the execution of this function as there is a check for that at the beginning of the function:
```
        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
```
Hence, the following line can be removed in both functions :
```
        collectionArtistPrimaryAddresses[_collectionID].status = false;
```
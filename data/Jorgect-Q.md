## Findings Summary

| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-WRONG-CALCULATION-IN-airDropTokens) | WRONG CALCULATION IN `airDropTokens`  | Low |
| [L-02](#l-02-SENSITIVE-PARAMTER-IN-A-COLLECTION-CAN-BE-CHANGED-DURING-MINTING-TIME) | SENSITIVE PARAMTER IN A COLLECTION CAN BE CHANGED DURING MINTING TIME  | Low |
| [L-03](#l-03-`mintAndAuction`-IS-NOT-CHECKING-IF-THE-PUBLIC-TIME-IS-DONE.) |  `mintAndAuction` IS NOT CHECKING IF THE PUBLIC TIME IS DONE.  | Low |
| [L-04](#l-04-IF-THE-ADDRESS-OF-THE-ARTIST-GET-COMPROMISED,-THAT-ADDRESS-GET-STUCK-IN-THE-PROTOCOL) | IF THE ADDRESS OF THE ARTIST GET COMPROMISED, THAT ADDRESS GET STUCK IN THE PROTOCOL  | Low |


## [L-01] WRONG CALCULATION IN airDropTokens 

The `collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;` is not taking in cosideration the tokens minted for the lasts recipient in the loop

```
 function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
        ...
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
    ----->  require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
            }
        }
    }
```

The `require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");` is just taking in consideration the actual index in the loop but is not taking in cosideration the past recipient in the array.

### Recomendation
Check the whole amount of token to mints, not just the actual recipients:

```
 function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
        ...
        uint256 tokensminted;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1 + tokensminted;
             tokensminted = _numberOfTokens[y];
    ----->  require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
            }
        }
    }
```



## [L-02] SENSITIVE PARAMTER IN A COLLECTION CAN BE CHANGED DURING MINTING TIME
Allow artist (which are not trusted entities due each collection has different artist) change sensitive parameters during the mint phase is not fair for users.

```
 function setCollectionCosts(
        uint256 _collectionID,
        uint256 _collectionMintCost,
        uint256 _collectionEndMintCost,
        uint256 _rate,
        uint256 _timePeriod,
        uint8 _salesOption,
        address _delAddress
    ) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        collectionPhases[_collectionID].collectionMintCost = _collectionMintCost; //@audit (low) sensitive parameter of a collection can be change meanwhile the minting is starting
        collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
        collectionPhases[_collectionID].rate = _rate;
        collectionPhases[_collectionID].timePeriod = _timePeriod;
        collectionPhases[_collectionID].salesOption = _salesOption;
        collectionPhases[_collectionID].delAddress = _delAddress;
        setMintingCosts[_collectionID] = true;
    }

 function setCollectionPhases(
        uint256 _collectionID,
        uint256 _allowlistStartTime,
        uint256 _allowlistEndTime,
        uint256 _publicStartTime,
        uint256 _publicEndTime,
        bytes32 _merkleRoot
    ) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
    }
```
[[Link]](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L157C5-L177C6)

### Recomendation:
Consider block this sensitive parameter when the minting is on going.

## [L-03] `mintAndAuction` IS NOT CHECKING IF THE PUBLIC TIME IS DONE.
The [[mintAndAuction]](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L276) function is not checking if the the time of the auction is done, miting after ` collectionPhases[_collectionID].publicEndTime`.

### Recomendation:
check if the ` collectionPhases[_collectionID].publicEndTime` is already done.

## [L-04] IF THE ADDRESS OF THE ARTIST GET COMPROMISED, THAT ADDRESS GET STUCK IN THE PROTOCOL.
If the address of the artist get compromised there is no way to changed, this can disrup the whole collection due that this artist can call sensitive function in the protocol And there is no way to change it.


### Recomendation:
Consider add a new function where admins or global admins can use to change the artist in the collection in case that the address get compromised `collectionAdditionalData[_collectionID].collectionArtistAddress`



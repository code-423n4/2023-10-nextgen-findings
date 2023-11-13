# GAS 

## Cache variables to save gas. 
1. In `NextGenCore.sol::createCollection()` cache `newCollectionIndex` to save gas. 
```
function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) external FunctionAdminRequired(this.createCollection.selector) {
  uint256 index = newCollectionIndex;
  collectionInfo[index].collectionName = _collectionName;
  collectionInfo[index].collectionArtist = _collectionArtist;
  collectionInfo[index].collectionDescription = _collectionDescription;
  collectionInfo[index].collectionWebsite = _collectionWebsite;
  collectionInfo[index].collectionLicense = _collectionLicense;
  collectionInfo[index].collectionBaseURI = _collectionBaseURI;
  collectionInfo[index].collectionLibrary = _collectionLibrary;
  collectionInfo[index].collectionScript = _collectionScript;
  isCollectionCreated[index] = true;
  newCollectionIndex = index + 1;
}
```
[NextGenCore.sol#L130C1-L141C6](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L130C1-L141C6)

2. In `MinterContract.sol::airDropTokens()` huge savings by caching constant
value calls: `gencore.viewTokensIndexMin(_collectionID)`, `gencore.viewCirSupply(_collectionID)` 
and `gencore.viewTokensIndexMax(_collectionID)`.  
```
function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
  require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
  uint256 collectionTokenMintIndex;
  uint256 tokensIndexMin = gencore.viewTokensIndexMin(_collectionID);
  uint256 tokensIndexMax = gencore.viewTokensIndexMax(_collectionID);
  for (uint256 y=0; y< _recipients.length; y++) {
    uint256 circSupply = gencore.viewCirSupply(_collectionID);
    collectionTokenMintIndex = tokensIndexMin + circSupply + _numberOfTokens[y] - 1;
    require(collectionTokenMintIndex <= tokensIndexMax, "No supply");
    for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
      uint256 mintIndex = tokensIndexMin + circSupply + i;
      gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
    }
  }
}

```
[MinterContract.sol#L181C1-L192C6](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L181C1-L192C6)

3. Unnecessary external call, reuse `collectionTokenMintIndex`.
`
uint256 mintIndex = collectionTokenMintIndex;
`
[MinterContract.sol#L281](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L281)


## Initialize structs to storage to save gas for more than one update. 
Ex: 
```
function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) external FunctionAdminRequired(this.createCollection.selector) {
  uint256 index = newCollectionIndex;
  collectionInfoStructure storage info = collectionInfo[index];
  info.collectionName = _collectionName;
  info.collectionArtist = _collectionArtist;
  info.collectionDescription = _collectionDescription;
  info.collectionWebsite = _collectionWebsite;
  info.collectionLicense = _collectionLicense;
  info.collectionBaseURI = _collectionBaseURI;
  info.collectionLibrary = _collectionLibrary;
  info.collectionScript = _collectionScript;
  isCollectionCreated[index] = true;
  newCollectionIndex = index + 1;
}
```

Instances: `setCollectionData()`, `createCollection()`, `addRandomizer()`, `airDropTokens()`,
  `mint()`, `burn()`, `updateCollectionInfo()`, `artistSignature()`, 

## Unnecessary sloads
  1. `newCollectionIndex = newCollectionIndex + 1` can be just `newCollectionIndex = 1` 
  as we know `newCollectionIndex` is 0.  
  [NextGenCore.solL#110](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110)


## Public functions not used from inside the contract can be declared external.
  1. `createCollection()`
  [NextGenCore.sol#L130](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L130)
  2. `setCollectionData()`
  [NextGenCore.sol#L147](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147)

## Redundant code
  1. Require check is not needed in `NextGenCore.sol::burn()` as it already checks
  for token ownership and as far as I've seen there's only a possibility to mint
  tokens between `reservedMinTokensIndex` and `reservedMaxTokensIndex`.
  [NextGenCore.sol#L206](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L206)



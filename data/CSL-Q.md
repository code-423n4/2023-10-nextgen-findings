## not critical[01] Meaningless packaging

When the random number requested by the contract to VRF completes the callback, the VRF contract will call the Core contract to set a random hash for the requested token.  

GitHub:[299](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L299)

```solidity
    File:smart-contracts/NextGenCore
    function setTokenHash(uint256 _collectionID, uint256 _mintIndex, bytes32 _hash) external;
```

But in the VRF contract _hash simply takes 32 bytes after packaging the requested random number and ID

GitHub:[66](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L66)

```solidity
    File:smart-contract/RandomizerVRF.sol
    gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
```

Packaging requestToToken[_requestId] seems meaningless because it only truncates 32 bytes, which is _RandomWords[0]

Moreover, the current VRF contract sets the number of random numbers returned in a single request to 1, but the manager can change this number. Assuming this number is later changed, the original interception method may prevent the returned random numbers from being fully used.  

  If want requestToToken[_requestId] to participate in random number operations, or if want more random numbers to participate, consider using keccak256 instead of bytes32().  

## not critical[02]Consider adding a randomizer Contract check before setting mint related data

After creating a collection and setting collection data through global management, collection management can set the relevant data for a mint (mint fee, sales model, mint stage) in the MinterContract.  

However, due to the need for relevant management to call another function (addRandomizer) to set the randomizerContract, it may not have been added at this time. Causing casting failures in the whitelist stage and public stage (because _mintProcessing cannot request random numbers).  

GitHub[157](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L157)

```solidity
     function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
```

GitHub[196](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L196)

```solidity
    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
```

This requires the manager to restart this mint stage to correct this error, so please consider adding a check to ensure that randomizerContract has been added before setting mint data.

## low[01] Duplicate or wrong assignment

### Lines of code

https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L380-L390

### Vulnerability details

The recurring of `require(collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved")` and `collectionArtistPrimaryAddresses[_collectionID].status = false;` appears redundant. This could be due to repetitive setting or might require adjustment to `true`.

```solidity
    function proposePrimaryAddressesAndPercentages(
        uint256 _collectionID, 
        address _primaryAdd1, 
        address _primaryAdd2, 
        address _primaryAdd3, 
        uint256 _add1Percentage, 
        uint256 _add2Percentage, 
        uint256 _add3Percentage
    ) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
        collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
        collectionArtistPrimaryAddresses[_collectionID].status = false;
    }
```

### Recommended Mitigation Steps

delete `collectionArtistPrimaryAddresses[_collectionID].status = false;` or change it to `true`.

## low[02] no return value

### Lines of code

https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L130-L141

### Vulnerability details

When creating a collection, not returning `newCollectionIndex` poses a problem: we are unable to determine the Index of the newly created collection. Ideally, upon creating a collection, obtaining the Index and informing the artist of their specific Index is essential. Without this index, artists cannot execute any contract calls. Moreover, in scenarios involving a substantial number of createCollection transactions, it becomes challenging to associate a particular Index with a specific collection.

```solidity
    function createCollection(
        string memory _collectionName, 
        string memory _collectionArtist, 
        string memory _collectionDescription, 
        string memory _collectionWebsite, 
        string memory _collectionLicense, 
        string memory _collectionBaseURI, 
        string memory _collectionLibrary, 
        string[] memory _collectionScript) 
    public FunctionAdminRequired(this.createCollection.selector) {
        collectionInfo[newCollectionIndex].collectionName = _collectionName;
        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
        isCollectionCreated[newCollectionIndex] = true;
        newCollectionIndex = newCollectionIndex + 1;
    }
```

### Recommended Mitigation Steps

```solidity
    function createCollection(
        string memory _collectionName, 
        string memory _collectionArtist, 
        string memory _collectionDescription, 
        string memory _collectionWebsite, 
        string memory _collectionLicense, 
        string memory _collectionBaseURI, 
        string memory _collectionLibrary, 
        string[] memory _collectionScript) 
    public FunctionAdminRequired(this.createCollection.selector) returns(uint256){
        collectionInfo[newCollectionIndex].collectionName = _collectionName;
        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
        isCollectionCreated[newCollectionIndex] = true;
        newCollectionIndex = newCollectionIndex + 1;
        return newCollectionIndex - 1;
    }
```

## 
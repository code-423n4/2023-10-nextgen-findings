# 1. Double storage storing leads to extensive gas usage

## Description

In the contracts `RandomizerNXT.sol`, `RandomizerVRF.sol` and `RandomizerRNG.sol` there is a `_gencore` parameter in the constructor, that is stored in two different storage slots: first in `gencore`, second in `gencoreContract` thus wasting a lot of gas.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L28-L29

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L42-L43

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L30-L31

```solidity
    constructor(address _gencore, address _adminsContract, address _arRNG) ArrngConsumer(_arRNG) {
        gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
        adminsContract = INextGenAdmins(_adminsContract);
    }
```

## Remediation

By removing one of these redundant storage variables, you will save on gas costs and storage space while maintaining the necessary functionality.

# 2. Slot Usage Optimization

## Description

The `createCollection` function in the `NextGenCore.sol` contract has been implemented in a gas-inefficient manner. Currently, it individually assigns multiple attributes to a collection by updating them one by one. This method is inefficient because it consumes more gas than necessary, especially when creating a collection with multiple attributes. The excessive gas consumption can lead to higher transaction costs and lower efficiency.

```solidity
    function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public FunctionAdminRequired(this.createCollection.selector) {
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

## Remediation

To optimize gas usage and reduce the contract's overall complexity, it's recommended to use a more gas-efficient approach. This can be achieved by creating a struct to represent a collection and initializing the struct with the provided attributes. Here's an example of how the function can be optimized:

```solidity
struct Collection {
    string collectionName;
    string collectionArtist;
    string collectionDescription;
    string collectionWebsite;
    string collectionLicense;
    string collectionBaseURI;
    string collectionLibrary;
    string[] collectionScript;
}

Collection public collections;  // Store collections in a mapping

function createCollection(
    string memory _collectionName,
    string memory _collectionArtist,
    string memory _collectionDescription,
    string memory _collectionWebsite,
    string memory _collectionLicense,
    string memory _collectionBaseURI,
    string memory _collectionLibrary,
    string[] memory _collectionScript
) public FunctionAdminRequired(this.createCollection.selector) {
    Collection memory newCollection = Collection(
        _collectionName,
        _collectionArtist,
        _collectionDescription,
        _collectionWebsite,
        _collectionLicense,
        _collectionBaseURI,
        _collectionLibrary,
        _collectionScript
    );
    collections[newCollectionIndex] = newCollection;
    isCollectionCreated[newCollectionIndex] = true;
    newCollectionIndex = newCollectionIndex + 1;
}
```
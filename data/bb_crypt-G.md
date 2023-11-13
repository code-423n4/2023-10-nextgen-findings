Gas optimisations in contract `NextGenCore`

1. Struct `collectionAdditionalDataStructure` contains `randomizerContract` and `randomizer`, which are storing the same address always. Recommandation -> delete one of them (randomizerContract)


2. Avoid using isCollectionCreated and instead create a field inside collectionInfoStructure or make a condition such as `collectionName` field can never be empty if a collection is created. 

3. Multiple mappings contains redundant information or can be grouped in a single mapping such:
	- `wereDataAdded` this mapping can be removed and a field can be added as a flag inside `collectionAdditonalDataStructure` 
	- 'isCollectionCreated' this mapping can be removed and a field can be added as a flag inside `collectionInfoStructure
	- `tokensMintedPerAddress` & `tokensMintedAllowlistAddress` & `tokensAirdropPerAddress` can be packed inside a single struct with packed variables (uint64) as a collection total supply can't be greater than 10000000000, as specified in `setcollectionData` -> `require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");`
	- `artistSigned` mapping can be removed is we make mandatory that artistSignatures can't be valid if they are empty

4. Cache storage variables in memory to save gas on reading and writing. At the end of all the modifications override the storage variable with the memory one. Example:
In function `createCollection` from:
```

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
to this:
```

    function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public FunctionAdminRequired(this.createCollection.selector) {
        collectionInfoStructure memory collection = collectionInfo[newCollectionIndex];
        collection.collectionName = _collectionName;
        collection.collectionArtist = _collectionArtist;
        collection.collectionDescription = _collectionDescription;
        collection.collectionWebsite = _collectionWebsite;
        collection.collectionLicense = _collectionLicense;
        collection.collectionBaseURI = _collectionBaseURI;
        collection.collectionLibrary = _collectionLibrary;
        collection.collectionScript = _collectionScript;
        collectionInfo[newCollectionIndex] = collection;
        isCollectionCreated[newCollectionIndex] = true;
        newCollectionIndex = newCollectionIndex + 1;
    }
```
This behaviour can be implemented in each function of the contracts
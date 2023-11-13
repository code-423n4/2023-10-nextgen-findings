# [NC] assigning `newCollectionIndex` in constructor

As Core contract is not an upgradable, assigning `newCollectionIndex` in constructor seems off, because the `newCollectionIndex = newCollectionIndex + 1;` is basically assigning `newCollectionIndex` to 1. We can just initialize the variable to 1, such as: `uint256 public newCollectionIndex = 1;`. Thus no need to add the assigning in the constructor.

```js
File: NextGenCore.sol
108:     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:         adminsContract = INextGenAdmins(_adminsContract);
110:         newCollectionIndex = newCollectionIndex + 1;
111:         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:     }
```

# [NC] Redundant variable in collectionAdditonalDataStructure struct

the randomizerContract and randomizer variables in the collectionAdditonalDataStructure struct basically the same. Consider to use one of them, when on use just cast the address to IRandomizer.

```js
File: NextGenCore.sol
44:     struct collectionAdditonalDataStructure {
45:         address collectionArtistAddress;
46:         uint256 maxCollectionPurchases;
47:         uint256 collectionCirculationSupply;
48:         uint256 collectionTotalSupply;
49:         uint256 reservedMinTokensIndex;
50:         uint256 reservedMaxTokensIndex;
51:         uint setFinalSupplyTimeAfterMint;
52:         address randomizerContract;
53:         IRandomizer randomizer;
54:     }
```

# [NC] Creating a collection but no returning the collectionId

Usually when we create an entry or record, the common practice is to return the `ID` of that created record (in this case collection). But if we look at `createCollection`, it's not returning any collectionId, which might be an issue if it's not being logged correctly. Meanwhile this `collectionId` will be use extensively in other functions.

```js
File: NextGenCore.sol
130:     function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public FunctionAdminRequired(this.createCollection.selector) {
```

# [NC] Passing struct as parameter

When creating a collection, instead of passing all struct `collectionInfoStructure`'s variable one by one, we ca just passing the struct parameter itself, constructing the struct offline before calling the function.

```js
File: NextGenCore.sol
130:     function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public FunctionAdminRequired(this.createCollection.selector) {
131:         collectionInfo[newCollectionIndex].collectionName = _collectionName;
132:         collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
133:         collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
134:         collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
135:         collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
136:         collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
137:         collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
138:         collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
139:         isCollectionCreated[newCollectionIndex] = true;
140:         newCollectionIndex = newCollectionIndex + 1;
141:     }
```

can be changed to 

```js
function createCollection(collectionInfoStructure memory _collectionInfo) public FunctionAdminRequired(this.createCollection.selector) {
    collectionInfo[newCollectionIndex] = _collectionInfo;
    isCollectionCreated[newCollectionIndex] = true;
    newCollectionIndex++;
}
```

This makes the function more readable and allows you to encapsulate the collection information in a single struct.



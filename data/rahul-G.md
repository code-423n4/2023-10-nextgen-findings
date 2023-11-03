### 976 gas can be saved during creation of collection

| **Method**       | NextGenCore:createCollection() |
|------------------|--------------------------------|
| **Before**       | 252555                         |
| **After**        | 251579 (-976)                  |
| **# Calls**      | 4                              |
| **Test passes?** | Yes                            |

Refactoring the creation of a collection, a critical entry point, into simpler code can save gas:

```diff
    function createCollection(__SNIP__) {
-        collectionInfo[newCollectionIndex].collectionName = _collectionName;
-        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
-        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
-        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
-        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
-        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
-        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
-        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
-        isCollectionCreated[newCollectionIndex] = true;
-        newCollectionIndex = newCollectionIndex + 1;
 
+       isCollectionCreated[newCollectionIndex] = true;
+       collectionInfo[newCollectionIndex++] = collectionInfoStructure(
+           _collectionName,
+           _collectionArtist,
+           _collectionDescription,
+           _collectionWebsite,
+           _collectionLicense,
+           _collectionBaseURI,
+           _collectionLibrary,
+           _collectionScript
+       );
	}
```

[📄 Source: smart-contracts/NextGenCore.sol#L130-L141](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L130-L141)
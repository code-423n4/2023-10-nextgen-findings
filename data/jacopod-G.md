Inside the function [`createCollection()`](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L130C14-L130C30) the storage variable `newCollectionIndex` is read multiple times from storage. Consider caching it into a new variable `_newCollectionIndex`, then update all the struct fields like:

    collectionInfo[_newCollectionIndex].collectionName = _collectionName;
    collectionInfo[_newCollectionIndex].collectionArtist = _collectionArtist;
    ...

and then perform the increment and save it to storage:

    _newCollectionIndex = _newCollectionIndex + 1;
    newCollectionIndex = _newCollectionIndex;


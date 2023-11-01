There isn't a need for combining Artist and Admin since you can add 2 modifers in a function `ArtistOrAdminRequired`. 
```
    // certain functions can only be called by an admin or the artist
    modifier ArtistOrAdminRequired(uint256 _collectionID, bytes4 _selector) {
      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
      _;
    }


    // certain functions can only be called by a global or function admin


    modifier FunctionAdminRequired(bytes4 _selector) {
      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
      _;
    }
```
It's a waste of gas deploying and using the `ArtistOrAdminRequired` for validation.

Instead just create another Modifer of `ArtistRequired` and use `FunctionAdminRequired`

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/MinterContract.sol#L143C1-L153C6
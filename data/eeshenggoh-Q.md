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

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/MinterContract.sol#L143C1-L153C6

It's a waste of gas deploying and using the `ArtistOrAdminRequired` for validation.

Instead, just create another Modifer of `ArtistRequired` and use `FunctionAdminRequired`



###########################
For this Admin contract `smart-contracts/NextGenAdmins.sol`. There is no event emitted when change of Admins, it is best practice to emit events on these functions.

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/NextGenAdmins.sol#L1C1-L87C2

Generally, events are used to inform the calling application about the current state of the contract, with the help of the logging facility of EVM. Events notify the applications about the change made to the contracts and applications which can be used to execute the dependent logic
======================================================================

For the function in MinterContract.sol, there is a wrong typo. Also do use custom error instead of require statements.
```
    function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        //@audit-issue English Typo
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
```
### [L-01] proposeSecondaryAddressesAndPercentages is not used

`proposeSecondaryAddressesAndPercentages()` can be called to propose royalty percentages to the artist, but it is not used in `payArtist()`. It is also not used anywhere else in the contract. The status of `collectionArtistSecondaryAddresses()` does not matter.

```
    function proposeSecondaryAddressesAndPercentages(uint256 _collectionID, address _secondaryAdd1, address _secondaryAdd2, address _secondaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposeSecondaryAddressesAndPercentages.selector) {
        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1 = _secondaryAdd1;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2 = _secondaryAdd2;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3 = _secondaryAdd3;
        collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
        collectionArtistSecondaryAddresses[_collectionID].status = false;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L394-L404

### [L-02] Non refundable msg.value will mess up the total collection amount

The mint functions in MinterContract are payable, so users can just put an appropriate msg.value inside. However, whatever value they put will not be refundable, and will be instead sent to the artists and team. Instead of accepting any msg.value that is higher than what is is required, only require that the msg.value is equal to the value of minting the NFT

```
        //@audit change the >= sign to == instead.
        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
```

Either calculate the exact amount off chain or refund the excess msg.value.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L233

### [L-03] setCollectionPhases start and end time is not checked properly

The ideal condition is for allowlistStartTime to be earlier than allowlistEndTime, and for the publicStartTime to be earlier than publicEndTime. Also, the allowlist start and end time should be earlier than the public start and end time. These checks are not forced when setting the collected phases, which may create some errors when minting later (for example if public time is set before allowlist time, then every one can mint instead of just allowing the allowlist to mint first)

```
    function setCollectionPhases(uint256 _collectionID, uint _allowlistStartTime, uint _allowlistEndTime, uint _publicStartTime, uint _publicEndTime, bytes32 _merkleRoot) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
    }
```

Do all the subsequent checks before setting the timings. Make sure that the timings are also greater than the current block.timestamp to prevent weird past minting errors.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L170-L178

### [L-04] There is no limit to the setting default royalties

The protocol accept royalties from all NFTs, and the royalty amount can be set by the admin in the NextGenCore.sol contract. (initially set to 6.9%). However, the admin can set a bps as high as 100% to get all the profits.

```
    function setDefaultRoyalties(address _royaltyAddress, uint96 _bps) public FunctionAdminRequired(this.setDefaultRoyalties.selector) {
        _setDefaultRoyalty(_royaltyAddress, _bps);
    }
```

Make sure to limit the royalty amount to protect against centralization risks.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L328

### [L-05] Circulating supply is quite misleading

In NextGenCore contract, the wording of `collectionCirculationSupply` is pretty misleading. When NFT is minted, circulating supply increases but when the NFT is burned, the circulating supply is not decreased.

```
    function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
```

Circulation technically means how many NFT are out there, so when an NFT is burned, the circulation supply should decrease by 1.

The more appropriate term to use should be mint, not circulate `collectionMintedSupply`.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L152

### [L-06] There is no point in calling burnToMint if users still have the pay the mint price

In burnToMint, a user burns an NFT in a collection to mint an NFT in a new collection. The user has to pay the full value of the NFT in the new collection. This does not make sense because the user can just call mint instead of calling burnToMint and wasting an NFT just to buy a new NFT.

```
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
```

Instead, there should be a discount if a user calls burnToMint. This is to incentivise users to call this function.

Also, check that \_burnCollectionId is not the same as \_mintCollectionId in `initializeBurn()`

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L258-L271

### [N-01] Line 389 and 403 of setting status to false is redundant

```
    function proposePrimaryAddressesAndPercentages(uint256 _collectionID, address _primaryAdd1, address _primaryAdd2, address _primaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
        collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
>       collectionArtistPrimaryAddresses[_collectionID].status = false;
    }
```

Since the first line in the function checks that the status must be false, there is no point setting the status to false again since it already is and will be false.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L380-L389

### [N-02] Misleading comments when salesOption == 3 in mint function

Comment states that users are able to mint after a day passes. This is only true if timePeriod is set to 1 days. In the test where timePeriod is set to 200 seconds, users are able to mint after 200 seconds, not after a day.

```
            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
            // users are able to mint after a day passes
            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L250
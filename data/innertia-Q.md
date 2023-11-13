## 1. Can change the collectionArtist even after signing with artistSignature.
As per the code below, the signature is locked, but `collectionArtist` can be still changed it afterwards. This is clearly undesirable.
If sign it, the artist should be locked up too.

 `collectionInfo[_collectionID].collectionArtist = _newCollectionArtist;`
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L242

```
    function artistSignature(uint256 _collectionID, string memory _signature) public {
        require(msg.sender == collectionAdditionalData[_collectionID].collectionArtistAddress, "Only artist");
        require(artistSigned[_collectionID] == false, "Already Signed");
        artistsSignatures[_collectionID] = _signature;
        artistSigned[_collectionID] = true;
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L257-L262

## 2. A malicious recipient can always make the airdrop fail
Since it is called inside `_mintProcessing` to the address of `_recipient`, a malicious recipient can always revert this because it can be a code that makes the recipient consume gas
You can stop using `_safemint`

```
    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L178C4-L185

## 3. Lack of validation of collection settings
Depending on how the items in the collection are set up, the function may always revert.
The following items should be properly validated.

Validation is needed because it should be set to `allowlistStartTime < allowlistEndTime < publicStartTime < publicEndTime`
```
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L172-L176

If `allowlistStartTime < timePeriod`, it will always be reverted, so it should be validated at setup time to avoid this.
`timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;`
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L244

` collectionPhases[_collectionID].timePeriod = _timePeriod;`
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L162

Also, from the following code, `timePeriod != 0`.
` uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;`
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L249C12-L249C12

Finally, from the code below, `collectionMintCost` must always be greater than or equal to `collectionEndMintCost`.
```
 if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L553

Please add validations that satisfy these
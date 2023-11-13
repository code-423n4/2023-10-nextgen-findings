# L-01 Use of `subscriptionId` while the owner/admin can be changed may be risky for funds of previous owner's `subId`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L56

```solidity
  function requestRandomWords(uint256 tokenid) public {
        require(msg.sender == gencore);
        uint256 requestId = COORDINATOR.requestRandomWords(
            keyHash,
            s_subscriptionId,
            requestConfirmations,
            callbackGasLimit,  
            numWords
        );
        tokenToRequest[tokenid] = requestId;
        requestToToken[requestId] = tokenid;
    }
```

The contract is ownable and has a `subscriptionId` setUp in the `requestRandomWords()` function, the same owner is responsible for maintaining a minimum amount of the LINK tokens with the same  `subscriptionId` to `requestRandomWords()`, In case the admin1 decides to transfer the ownership to admin2 the `subscriptionId` will not be changed in that case, only the ownership of the contract is changed.

### Before transferring the ownership
Contract owner - admin1, subscriptionId of : admin1

### After transferring the ownership
Contract owner - admin2, subscriptionId of : admin1

In this situation the old admin won't be aware of his funds being used for calling `requestRandomWords()` while the new admin keeps on calling it with the funds of old admin i.e `admin1`.

The question arises if the admins are from the same team? I think there could be possibility of admins being from the different teams or the ownership being handled by the third party or so.


# L-02 - Check-Effect-Interaction pattern is not followed and Re-entrancy is possible

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178C4-L185C6

```solidity
 function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }
```

# L-03 `burnToMint()` fucntion is using `_burn()` but not checking if valid `_tokenId` is provided.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L213-L224

While the function `_burn()` has a requirement to check that `tokenId` must exist. but in the function `burnToMint()` there is no check in place to ensure the existence of a valid `tokenId` as a parameter.

```solidity
 function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
            // burn token
            _burn(_tokenId);
            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
        }
    }
```
```diff

+  require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");
```






# G-01 Use `x += 1` instead of using `x = x+1`

There are many instances in the code where the above pattern is not followed, one such instance is :

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L180

```solidity
    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }
```
## Use this pattern instead.
```diff
- collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
+ collectionAdditionalData[_collectionID].collectionCirculationSupply+= 1 ;
```
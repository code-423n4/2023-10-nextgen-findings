# QA Report

## Summary

### Low Issues

Total **29 instances** over **5 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[L&#x2011;01]](#l01-multiplication-on-the-result-of-a-division) | Multiplication on the result of a division | 2 |
| [[L&#x2011;02]](#l02-nft-doesnt-handle-hard-forks) | NFT doesn't handle hard forks | 1 |
| [[L&#x2011;03]](#l03-external-call-recipient-can-consume-all-remaining-gas) | External call recipient can consume all remaining gas | 11 |
| [[L&#x2011;04]](#l04-consider-implementing-two-step-procedure-for-updating-protocol-addresses) | Consider implementing two-step procedure for updating protocol addresses | 14 |
| [[L&#x2011;05]](#l05-the-remaining-eth-may-be-locked-in-the-contract-after-call) | The remaining ETH may be locked in the contract after call | 1 |

### Non Critical Issues

Total **117 instances** over **10 issues**:

|ID|Issue|Instances|
|:--:|:---|:--:|
| [[N&#x2011;01]](#n01-abiencodepacked-on-strings-should-be-replaced-with-stringconcat) | `abi.encodePacked()` on strings should be replaced with `string.concat()` | 7 |
| [[N&#x2011;02]](#n02-simplify-complex-require-statements) | Simplify complex require statements | 7 |
| [[N&#x2011;03]](#n03-missing-checks-for-empty-bytes-when-updating-bytes-state-variables) | Missing checks for empty bytes when updating bytes state variables | 3 |
| [[N&#x2011;04]](#n04-constructor-visibility-specifier-should-be-omitted) | Constructor visibility specifier should be omitted | 1 |
| [[N&#x2011;05]](#n05-contract-name-does-not-match-its-filename) | Contract name does not match its filename | 6 |
| [[N&#x2011;06]](#n06-empty-function-body-without-comments) | Empty function body without comments | 1 |
| [[N&#x2011;07]](#n07-event-is-missing-indexed-fields) | Event is missing `indexed` fields | 4 |
| [[N&#x2011;08]](#n08-missing-zero-address-check-in-functions-with-address-parameters) | Missing zero address check in functions with address parameters | 32 |
| [[N&#x2011;09]](#n09-whitespace-in-expressions) | Whitespace in Expressions | 19 |
| [[N&#x2011;10]](#n10-error-messages-should-be-descriptive-not-cryptic) | Error messages should be descriptive, not cryptic | 37 |

## Low Issues

### [L&#x2011;01] Multiplication on the result of a division

Dividing a number by another often results in a loss of precision. Multiplying the result by another number increases the loss of precision, often significantly.
For example: `x*(y/z)` should be re-written as `(x*z)/y`.

There are 2 instances:

- *MinterContract.sol* ( [536](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L536), [551](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L551) ):

```solidity
536:                 return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));

551:                 decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
```

### [L&#x2011;02] NFT doesn't handle hard forks

When there are hard forks, users often have to go through [many hoops](https://twitter.com/elerium115/status/1558471934924431363) to ensure that they control ownership on every fork. Consider adding `require(1 == chain.chainId)`, or the chain ID of whichever chain you prefer, to the functions below, or at least include the chain ID in the URI, so that there is no confusion about which chain is the owner of the NFT.

There is 1 instance:

- *NextGenCore.sol* ( [343-357](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L343-L357) ):

```solidity
343:     function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
344:         _requireMinted(tokenId);
345:         if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
346:             string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
347:             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
348:         } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
349:             string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
350:             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
351:         }
352:         else {
353:             string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));
354:             string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));
355:             return _uri;
356:         }
357:     }
```

### [L&#x2011;03] External call recipient can consume all remaining gas

There is no limit specified on the amount of gas used, so the recipient can use up all of the remaining gas(`gasleft()`), causing it to revert. Therefore, when calling an external contract, it is necessary to specify a limited amount of gas to forward.

There are 11 instances:

- *AuctionDemo.sol* ( [113](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L113), [116](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L116), [128](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L128), [139](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L139) ):

```solidity
113:                 (bool success, ) = payable(owner()).call{value: highestBid}("");

116:                 (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");

128:         (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");

139:                 (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```

- *MinterContract.sol* ( [434](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L434), [435](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L435), [436](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L436), [437](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L437), [438](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L438), [464](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L464) ):

```solidity
434:         (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");

435:         (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");

436:         (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");

437:         (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");

438:         (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");

464:         (bool success, ) = payable(admin).call{value: balance}("");
```

- *RandomizerRNG.sol* ( [82](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L82) ):

```solidity
82:         (bool success, ) = payable(admin).call{value: balance}("");
```

### [L&#x2011;04] Consider implementing two-step procedure for updating protocol addresses

A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending, and must 'accept' the assignment by making an affirmative call. A straight forward way of doing this would be to have the target contracts implement [EIP-165](https://eips.ethereum.org/EIPS/eip-165), and to have the 'set' functions ensure that the recipient is of the right interface type.

There are 14 instances:

- *MinterContract.sol* ( [157-166](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L157-L166), [302-304](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L302-L304), [448-450](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L448-L450), [454-457](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L454-L457) ):

```solidity
157:     function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
158:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
159:         collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
160:         collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
161:         collectionPhases[_collectionID].rate = _rate;
162:         collectionPhases[_collectionID].timePeriod = _timePeriod;
163:         collectionPhases[_collectionID].salesOption = _salesOption;
164:         collectionPhases[_collectionID].delAddress = _delAddress;
165:         setMintingCosts[_collectionID] = true;
166:     }

302:     function updateDelegationCollection(uint256 _collectionID, address _collectionAddress) public FunctionAdminRequired(this.updateDelegationCollection.selector) { 
303:         collectionPhases[_collectionID].delAddress = _collectionAddress;
304:     }

448:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
449:         gencore = INextGenCore(_gencore);
450:     }

454:     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
455:         require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
456:         adminsContract = INextGenAdmins(_newadminsContract);
457:     }
```

- *NextGenCore.sol* ( [147-166](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147-L166), [322-325](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L322-L325), [329-331](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L329-L331) ):

```solidity
147:     function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
148:         require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
149:         if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
150:             collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
151:             collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
152:             collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
153:             collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
154:             collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
155:             collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
156:             collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
157:             wereDataAdded[_collectionID] = true;
158:         } else if (artistSigned[_collectionID] == false) {
159:             collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
160:             collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
161:             collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
162:         } else {
163:             collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
164:             collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
165:         }
166:     }

322:     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
323:         require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
324:         adminsContract = INextGenAdmins(_newadminsContract);
325:     }

329:     function setDefaultRoyalties(address _royaltyAddress, uint96 _bps) public FunctionAdminRequired(this.setDefaultRoyalties.selector) {
330:         _setDefaultRoyalty(_royaltyAddress, _bps);
331:     }
```

- *RandomizerNXT.sol* ( [41-43](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L41-L43), [45-47](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L45-L47), [49-52](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L49-L52) ):

```solidity
41:     function updateRandomsContract(address _randoms) public FunctionAdminRequired(this.updateRandomsContract.selector) {
42:         randoms = IXRandoms(_randoms);
43:     }

45:     function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {
46:         adminsContract = INextGenAdmins(_admin);
47:     }

49:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
50:         gencore = _gencore;
51:         gencoreContract = INextGenCore(_gencore);
52:     }
```

- *RandomizerRNG.sol* ( [61-64](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L61-L64), [66-69](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L66-L69) ):

```solidity
61:     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
62:         require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
63:         adminsContract = INextGenAdmins(_newadminsContract);
64:     }

66:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
67:         gencore = _gencore;
68:         gencoreContract = INextGenCore(_gencore);
69:     }
```

- *RandomizerVRF.sol* ( [94-97](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L94-L97), [99-102](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L99-L102) ):

```solidity
94:     function updateAdminContract(address _newadminsContract) public FunctionAdminRequired(this.updateAdminContract.selector) {
95:         require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
96:         adminsContract = INextGenAdmins(_newadminsContract);
97:     }

99:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
100:         gencore = _gencore;
101:         gencoreContract = INextGenCore(_gencore);
102:     }
```

### [L&#x2011;05] The remaining ETH may be locked in the contract after call

After calling an external contract and forwards some ETH value, the contract balance should be checked. If there is excess eth left over due to a failed call, or more eth being delivered than needed, or any other reason, this eth must be refunded to the user or handled appropriately, otherwise the eth may be frozen in the contract forever.

There is 1 instance:

- *RandomizerRNG.sol* ( [42](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L42) ):

```solidity
/// @audit requestRandomWords()
42:         uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));
```

## Non Critical Issues

### [N&#x2011;01] `abi.encodePacked()` on strings should be replaced with `string.concat()`

Solidity version 0.8.12 introduces `string.concat()`, which can be used to replace `abi.encodePacked()` on strings and makes the intended operation clearer, leading to less reviewer confusion.

There are 7 instances:

- *NextGenCore.sol* ( [347](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L347), [350](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L350), [353](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L353), [354](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L354), [363](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L363), [454](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L454), [456](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L456) ):

```solidity
347:             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

350:             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";

353:             string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));

354:             string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));

363:         return string(abi.encodePacked(collectionInfo[viewColIDforTokenID(tokenId)].collectionName, " #" ,tok.toString()));

454:             scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 

456:         return string(abi.encodePacked("let hash='",Strings.toHexString(uint256(tokenToHash[tokenId]), 32),"';let tokenId=",tokenId.toString(),";let tokenData=[",tokenData[tokenId],"];", scripttext));
```

### [N&#x2011;02] Simplify complex require statements

Simplifying complex `require` statements with local variables and `if`(or `revert`) statements can improve readability, make debugging easier, and promote modularity and reusability in the code.

There are 7 instances:

- *AuctionDemo.sol* ( [32](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L32), [58](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L58), [105](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L105) ):

```solidity
32:       require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

58:         require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);

105:         require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
```

- *MinterContract.sol* ( [137](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L137), [151](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L151) ):

```solidity
137:       require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

151:       require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
```

- *NextGenCore.sol* ( [124](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L124), [148](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148) ):

```solidity
124:       require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

148:         require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
```

### [N&#x2011;03] Missing checks for empty bytes when updating bytes state variables

Unless the code is attempting to `delete` the state variable, the caller shouldn't have to write `""` to the state variable

There are 3 instances:

- *MinterContract.sol* ( [174](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L174) ):

```solidity
174:         collectionPhases[_collectionID].merkleRoot = _merkleRoot;
```

- *NextGenCore.sol* ( [302](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L302) ):

```solidity
302:         tokenToHash[_mintIndex] = _hash;
```

- *RandomizerVRF.sol* ( [81](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L81) ):

```solidity
81:         keyHash = _keyHash;
```

### [N&#x2011;04] Constructor visibility specifier should be omitted

Visibility for constructor is ignored, so the visibility specifier should be omitted to avoid misunderstanding. If you want the contract to be non-deployable, making it "abstract" is sufficient.

There is 1 instance:

- *AuctionDemo.sol* ( [36](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L36) ):

```solidity
36:     constructor (address _minter, address _gencore, address _adminsContract) public {
```

### [N&#x2011;05] Contract name does not match its filename

According to the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html#contract-and-library-names), contract and library names should also match their filenames.

There are 6 instances:

- *AuctionDemo.sol* ( [18](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L18) ):

```solidity
/// @audit Not match filename `AuctionDemo.sol`
18: contract auctionDemo is Ownable {
```

- *MinterContract.sol* ( [20](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L20) ):

```solidity
/// @audit Not match filename `MinterContract.sol`
20: contract NextGenMinterContract is Ownable {
```

- *RandomizerNXT.sol* ( [18](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L18) ):

```solidity
/// @audit Not match filename `RandomizerNXT.sol`
18: contract NextGenRandomizerNXT {
```

- *RandomizerRNG.sol* ( [18](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L18) ):

```solidity
/// @audit Not match filename `RandomizerRNG.sol`
18: contract NextGenRandomizerRNG is ArrngConsumer, Ownable {
```

- *RandomizerVRF.sol* ( [19](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L19) ):

```solidity
/// @audit Not match filename `RandomizerVRF.sol`
19: contract NextGenRandomizerVRF is VRFConsumerBaseV2, Ownable {
```

- *XRandoms.sol* ( [13](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L13) ):

```solidity
/// @audit Not match filename `XRandoms.sol`
13: contract randomPool {
```

### [N&#x2011;06] Empty function body without comments

Empty function body in solidity is not recommended, consider adding some comments to the body.

There is 1 instance:

- *RandomizerRNG.sol* ( [86](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L86) ):

```solidity
86:     receive() external payable {}
```

### [N&#x2011;07] Event is missing `indexed` fields

Index event fields makes the field more quickly accessible to [off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. However, note that each indexed field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

There are 4 instances:

- *MinterContract.sol* ( [124](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L124), [125](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L125), [126](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L126) ):

```solidity
124:     event PayArtist(address indexed _add, bool status, uint256 indexed funds);

125:     event PayTeam(address indexed _add, bool status, uint256 indexed funds);

126:     event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

- *RandomizerRNG.sol* ( [24](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L24) ):

```solidity
24:     event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```

### [N&#x2011;08] Missing zero address check in functions with address parameters

Adding a zero address check for each address type parameter can prevent errors.

There are 32 instances:

- *MinterContract.sol* ( [157](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L157), [181](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181), [196](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L196), [276](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L276), [302](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L302), [315](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L315), [326](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L326), [380](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L380), [394](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L394), [415](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L415), [448](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L448) ):

```solidity
/// @audit `_delAddress not checked`
157:     function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {

/// @audit `_recipients not checked`
181:     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {

/// @audit `_mintTo not checked`
196:     function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {

/// @audit `_recipient not checked`
276:     function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {

/// @audit `_collectionAddress not checked`
302:     function updateDelegationCollection(uint256 _collectionID, address _collectionAddress) public FunctionAdminRequired(this.updateDelegationCollection.selector) { 

/// @audit `_erc721Collection not checked`
/// @audit `_burnOrSwapAddress not checked`
315:     function initializeExternalBurnOrSwap(address _erc721Collection, uint256 _burnCollectionID, uint256 _mintCollectionID, uint256 _tokmin, uint256 _tokmax, address _burnOrSwapAddress, bool _status) public FunctionAdminRequired(this.initializeExternalBurnOrSwap.selector) { 

/// @audit `_erc721Collection not checked`
326:     function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {

/// @audit `_primaryAdd1 not checked`
/// @audit `_primaryAdd2 not checked`
/// @audit `_primaryAdd3 not checked`
380:     function proposePrimaryAddressesAndPercentages(uint256 _collectionID, address _primaryAdd1, address _primaryAdd2, address _primaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {

/// @audit `_secondaryAdd1 not checked`
/// @audit `_secondaryAdd2 not checked`
/// @audit `_secondaryAdd3 not checked`
394:     function proposeSecondaryAddressesAndPercentages(uint256 _collectionID, address _secondaryAdd1, address _secondaryAdd2, address _secondaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposeSecondaryAddressesAndPercentages.selector) {

/// @audit `_team1 not checked`
/// @audit `_team2 not checked`
415:     function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {

/// @audit `_gencore not checked`
448:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
```

- *NextGenAdmins.sol* ( [38](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L38), [44](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L44), [50](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L50), [58](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L58) ):

```solidity
/// @audit `_admin not checked`
38:     function registerAdmin(address _admin, bool _status) public onlyOwner {

/// @audit `_address not checked`
44:     function registerFunctionAdmin(address _address, bytes4 _selector, bool _status) public AdminRequired {

/// @audit `_address not checked`
50:     function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {

/// @audit `_address not checked`
58:     function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {
```

- *NextGenCore.sol* ( [147](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147), [178](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L178), [189](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L189), [213](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L213), [329](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L329) ):

```solidity
/// @audit `_collectionArtistAddress not checked`
147:     function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {

/// @audit `_recipient not checked`
178:     function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {

/// @audit `_mintingAddress not checked`
/// @audit `_mintTo not checked`
189:     function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {

/// @audit `burner not checked`
213:     function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {

/// @audit `_royaltyAddress not checked`
329:     function setDefaultRoyalties(address _royaltyAddress, uint96 _bps) public FunctionAdminRequired(this.setDefaultRoyalties.selector) {
```

- *RandomizerNXT.sol* ( [41](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L41), [45](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L45), [49](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L49) ):

```solidity
/// @audit `_randoms not checked`
41:     function updateRandomsContract(address _randoms) public FunctionAdminRequired(this.updateRandomsContract.selector) {

/// @audit `_admin not checked`
45:     function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {

/// @audit `_gencore not checked`
49:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
```

- *RandomizerRNG.sol* ( [66](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L66) ):

```solidity
/// @audit `_gencore not checked`
66:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
```

- *RandomizerVRF.sol* ( [99](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L99) ):

```solidity
/// @audit `_gencore not checked`
99:     function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
```

### [N&#x2011;09] Whitespace in Expressions

See the [Whitespace in Expressions](https://docs.soliditylang.org/en/latest/style-guide.html#whitespace-in-expressions) section of the Solidity Style Guide.

There are 19 instances:

- *AuctionDemo.sol* ( [113](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L113), [116](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L116), [128](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L128), [139](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L139) ):

```solidity
/// @audit Whitespace inside parenthesis
113:                 (bool success, ) = payable(owner()).call{value: highestBid}("");

/// @audit Whitespace inside parenthesis
116:                 (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");

/// @audit Whitespace inside parenthesis
128:         (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");

/// @audit Whitespace inside parenthesis
139:                 (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```

- *MinterContract.sol* ( [144](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L144), [246](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L246), [289](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L289), [434](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L434), [435](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L435), [436](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L436), [437](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L437), [438](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L438), [464](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L464) ):

```solidity
/// @audit Whitespace before a comma
144:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

/// @audit More than one space around an operator
246:                 timeOfLastMint =  lastMintDate[col];

/// @audit More than one space around an operator
289:             timeOfLastMint =  lastMintDate[_collectionID];

/// @audit Whitespace inside parenthesis
434:         (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");

/// @audit Whitespace inside parenthesis
435:         (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");

/// @audit Whitespace inside parenthesis
436:         (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");

/// @audit Whitespace inside parenthesis
437:         (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");

/// @audit Whitespace inside parenthesis
438:         (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");

/// @audit Whitespace inside parenthesis
464:         (bool success, ) = payable(admin).call{value: balance}("");
```

- *NextGenCore.sol* ( [117](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L117), [189](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L189), [363](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L363) ):

```solidity
/// @audit Whitespace before a comma
117:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

/// @audit Whitespace before a comma
189:     function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {

/// @audit Whitespace before a comma
363:         return string(abi.encodePacked(collectionInfo[viewColIDforTokenID(tokenId)].collectionName, " #" ,tok.toString()));
```

- *RandomizerNXT.sol* ( [35](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L35) ):

```solidity
/// @audit Whitespace before a comma
35:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```

- *RandomizerRNG.sol* ( [82](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L82) ):

```solidity
/// @audit Whitespace inside parenthesis
82:         (bool success, ) = payable(admin).call{value: balance}("");
```

- *RandomizerVRF.sol* ( [48](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L48) ):

```solidity
/// @audit Whitespace before a comma
48:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```

### [N&#x2011;10] Error messages should be descriptive, not cryptic

Consider make these error messages more descriptive.

There are 37 instances:

- *AuctionDemo.sol* ( [32](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L32) ):

```solidity
32:       require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
```

- *MinterContract.sol* ( [137](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L137), [144](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L144), [151](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L151), [158](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L158), [182](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L182), [186](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L186), [211](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L211), [213](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L213), [217](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L217), [224](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L224), [228](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L228), [232](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L232), [233](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L233), [260](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L260), [265](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L265), [266](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L266), [277](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L277), [280](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L280), [309](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L309), [317](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L317), [337](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L337), [356](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L356), [360](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L360), [361](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L361), [382](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L382), [396](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L396) ):

```solidity
137:       require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

144:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

151:       require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

158:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

182:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

186:             require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

211:                 require(isAllowedToMint == true, "No delegation");

213:                 require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");

217:                 require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, msg.sender) + _numberOfTokens, "AL limit");

224:             require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");

228:             revert("No minting");

232:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

233:         require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");

260:         require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");

265:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");

266:         require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");

277:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

280:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

309:         require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

317:         require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

337:             require(isAllowedToMint == true, "No delegation");

356:             revert("No minting");

360:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

361:         require(msg.value >= (getPrice(col) * 1), "Wrong ETH");

382:         require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");

396:         require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
```

- *NextGenAdmins.sol* ( [32](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L32) ):

```solidity
32:       require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
```

- *NextGenCore.sol* ( [117](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L117), [124](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L124), [206](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L206), [239](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L239), [267](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L267), [293](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L293) ):

```solidity
117:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

124:       require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

206:         require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");

239:         require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

267:         require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

293:         require(isCollectionCreated[_collectionID] == true, "No Col");
```

- *RandomizerNXT.sol* ( [35](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L35) ):

```solidity
35:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```

- *RandomizerRNG.sol* ( [36](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L36) ):

```solidity
36:         require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
```

- *RandomizerVRF.sol* ( [48](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L48) ):

```solidity
48:       require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```


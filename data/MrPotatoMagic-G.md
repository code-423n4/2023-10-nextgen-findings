# Gas Optimization Report

| Gas Optimizations | Issues                                                                                                      | Instances |
|-------------------|-------------------------------------------------------------------------------------------------------------|-----------|
| [G-01]            | Use do-while loops instead of for loops to save gas                                                         | 3         |
| [G-02]            | Variable only used in current contract can be marked private to save gas                                    | 6         |
| [G-03]            | Update `newCollectionIndex` to 1 directly in the constructor to save gas                                    | 1         |
| [G-04]            | Use `++newCollectionIndex` instead of `newCollectionIndex = newCollectionIndex + 1` to save gas             | 1         |
| [G-05]            | Allowlist/Public start and end times can use smaller uint sizes in struct `collectionPhasesDataStructure`   | 1         |
| [G-06]            | Redundant check in public phase conditional block present in `mint()` function can be removed to save gas   | 1         |
| [G-07]            | No need to calculate `mintIndex` since `collectionTokenMintIndex` holds the same value                      | 1         |
| [G-08]            | Add address(0) checks to prevent unnecessary external calls to primary addresses that are address(0)        | 1         |
| [G-09]            | Simplify mathematical equation for `decreaserate` in Exponential Decay function (sales model 2) to save gas | 1         |
| [G-10]            | Return early when token reaches resting price in Linear Descending Sale (sales model 2) to save gas         | 1         |
| [G-11]            | No need to track variable `highBid` in for loop of function `returnHighestBid()` to save gas                | 1         |
| [G-12]            | Remove redundant check from function `returnHighestBidder()` to save gas                                    | 1         |
| [G-13]            | Use `msg.sender` instead of accessing bidder from storage in function `cancelBid()` to save gas             | 1         |

### Total Deployment cost saved: 169935 gas saved

### Total Function execution cost saved: 2140 gas saved (minimum per call)

### Total Gas saved: 172075 gas saved

## [G-01] Use do-while loops instead of for loops to save gas

There are 3 instances of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L50C1-L54C6)

**Before VS After**

**Deployment cost: 582355 - 571365 = 10990 gas saved**

Instead of this:
```solidity
File: NextGenAdmins.sol
50:     function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
51:         //@audit Gas - Use do-while loop
52:         for (uint256 i=0; i<_selector.length; i++) { 
53:             functionAdmin[_address][_selector[i]] = _status;
54:         }
55:     }
```
Use this:
```solidity
File: NextGenAdmins.sol
50:     function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
51:         /* for (uint256 i=0; i<_selector.length; i++) {
52:             functionAdmin[_address][_selector[i]] = _status;
53:         } */
54:         uint256 i;
55:         do {
56:             functionAdmin[_address][_selector[i]] = _status;
57:             unchecked {
58:                 ++i;
59:             }
60:         } while (i < _selector.length);
61:     }
```

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L281C1-L288C6)

**Before VS After**

**Deployment cost: 5501759 - 5499141 = 2618 gas saved**

Instead of this:
```solidity
File: NextGenCore.sol
289:     function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
290:         for (uint256 x; x < _tokenId.length; x++) {
292:             require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
293:             _requireMinted(_tokenId[x]);
294:             tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
295:             tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
296:         }
297:     }
```
Use this:
```solidity
File: NextGenCore.sol
283:     function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
290:         uint256 x;
291:         do {
292:             require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
293:             _requireMinted(_tokenId[x]);
294:             tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
295:             tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
296:             unchecked {
297:                 ++x;
298:             }
299:         } while(x < _tokenId.length);
300:     }
```

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L184)

**Before VS After**

**Deployment cost: 5454331 - 5448464 = 5867 gas saved**

**Function execution cost: 744940 - 744379 = 561 gas saved (per call)**

Instead of this:
```solidity
File: MinterContract.sol
184:     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
185:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
186:         uint256 collectionTokenMintIndex;
187:         for (uint256 y=0; y< _recipients.length; y++) {
190:             collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
191:             require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
192:             for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
193:                 uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
194:                 gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
195:             }
196:         }
197:     }
```
Use this:
```solidity
File: MinterContract.sol
181:     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
182:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
183:         uint256 collectionTokenMintIndex;
192:         uint256 y;
193:         do {
194:             collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
195:             require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
196:             uint256 i;
197:             do {
198:                 uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
199:                 gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
200:                 unchecked {
201:                     ++i;
202:                 }
203:             } while(i < _numberOfTokens[y]);
204:             unchecked {
205:                 ++y;
206:             }
207:         } while(y < _recipients.length);
208:     }
```

## [G-02] Variable only used in current contract can be marked private to save gas

There are 6 instances of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L26)

**Before VS After**

**Deployment cost: 5501759 - 5497380 = 4379 gas saved**

```solidity
File: NextGenCore.sol
26:   uint256 public newCollectionIndex;
```

[Link to remaining 5 instances below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L83C1-L105C35)

**Before VS After**

**Deployment cost: 5501759 - 5454222 = 47537 gas saved**

```solidity
File: NextGenCore.sol
082:     // current amount of burnt tokens per collection
083:     mapping (uint256 => uint256) private burnAmount;
084: 
085:     // modify the metadata view
086:     mapping (uint256 => bool) private onchainMetadata; 
087: 
088:     // artist signature per collection
089:     mapping (uint256 => string) private artistsSignatures;
099: 
100:     // artist signed
101:     mapping (uint256 => bool) private artistSigned; 
102:
105:     address private minterContract;
```

## [G-03] Update `newCollectionIndex` to 1 directly in the constructor to save gas

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L110)

**Before VS After**

**Deployment cost: 5501759 - 5500900 = 859 gas saved**

The state variable `newCollectionIndex` is initially 0 and incremented to 1 using the `x = x + y` pattern. This is done to ensure collectionIds do not start with 0. But instead of using the `x = x + y` pattern, we can just simply update `newCollectionIndex` to 1 to save gas.

Instead of this:
```solidity
File: NextGenCore.sol
108:     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:         adminsContract = INextGenAdmins(_adminsContract); 
110:         newCollectionIndex = newCollectionIndex + 1;
111:         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:     }
```
Use this:
```solidity
File: NextGenCore.sol
108:     constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:         adminsContract = INextGenAdmins(_adminsContract); 
110:         newCollectionIndex = 1;
111:         _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:     }
```

## [G-04] Use `++newCollectionIndex` instead of `newCollectionIndex = newCollectionIndex + 1` to save gas

There is 1 instance of this:

**Before VS After**

**Deployment cost: 5501759 - 5501291 = 468 gas saved**

**Function execution cost: 252555 - 252543 = 12 gas saved (per call)**

Instead of this:
```solidity
File: NextGenCore.sol
141:     newCollectionIndex = newCollectionIndex + 1;
```
Use this:
```solidity
File: NextGenCore.sol
141:     ++newCollectionIndex;
```

## [G-05] Allowlist/Public start and end times can use smaller uint sizes in struct `collectionPhasesDataStructure`

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L44C1-L56C6)

Currently the first 4 members of the collectionPhasesDataStructure occupy 4 slots. These members are time related and can make use of smaller uint sizes such as uint64 to save slots and gas. 

If we use uint64 (8 bytes) as the size for each of the first 4 members, we can fit all of them in 1 slot (8 * 4 = 32 bytes = 1 slot) instead of 4 slots. This can save gas whenever read/write operations are performed on a struct instance.

Instead of this:
```solidity
File: MinterContract.sol
46:     struct collectionPhasesDataStructure {
47:         uint allowlistStartTime; //occupy 4 slots
48:         uint allowlistEndTime;
49:         uint publicStartTime; 
50:         uint publicEndTime;
51:         bytes32 merkleRoot;
52:         uint256 collectionMintCost;
53:         uint256 collectionEndMintCost;
54:         uint256 timePeriod;
55:         uint256 rate;
56:         uint8 salesOption;
57:         address delAddress;
58:     }
```
Use this:
```solidity
File: MinterContract.sol
46:     struct collectionPhasesDataStructure {
47:         uint64 allowlistStartTime; //now occupy 1 slot
48:         uint64 allowlistEndTime;
49:         uint64 publicStartTime; 
50:         uint64 publicEndTime;
51:         bytes32 merkleRoot;
52:         uint256 collectionMintCost;
53:         uint256 collectionEndMintCost;
54:         uint256 timePeriod;
55:         uint256 rate;
56:         uint8 salesOption;
57:         address delAddress;
58:  
```

## [G-06] Redundant check in public phase conditional block present in `mint()` function can be removed to save gas 

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L223)

**Before VS After**

**Deployment cost: 5454331 - 5415855 = 38476 gas saved**

**Function execution cost: 531882 - 530831 = 1051 gas saved (per call)**

The conditional else-if block below is from the [mint()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L196) function. The block is used when in public phase. On Line 240, we can remove the require check since on the following Line 241, the condition is already covered in the require check. Thus, this makes the check redundant on Line 240 and should be removed.
```solidity
File: MinterContract.sol
238:         } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
239:             phase = 2;
240:             require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
241:             require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
242:             mintingAddress = msg.sender;
243:             tokData = '"public"';
```

## [G-07] No need to calculate `mintIndex` since `collectionTokenMintIndex` holds the same value 

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L263C1-L267C118)

**Before VS After**

**Deployment cost: 5454331 - 5403794 = 50537**

On Line 281 and 284, we can observe `mintIndex` retrieving the same value that was previously retrieved by `collectionTokenMintIndex`. Thus, there is no need to create a mintIndex variable and the value of collectionTokenMintIndex can be directly used as the tokenId for the call to the NextGenCore contract on Line 287.

Instead of this:
```solidity
File: MinterContract.sol
275:     function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
276:         require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
277:         require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
278:         require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
279:         // minting new token
280:         uint256 collectionTokenMintIndex;
281:         collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
282:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
283:         require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
284:         uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
285:         // burn and mint token
286:         address burner = msg.sender;
287:         gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
288:         collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
289:     }
```
Use this:
```solidity
File: MinterContract.sol
275:     function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
276:         require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
277:         require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
278:         require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
279:         // minting new token
280:         uint256 collectionTokenMintIndex;
281:         collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
282:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
283:         require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
284:         //uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
285:         // burn and mint token
286:         address burner = msg.sender;
287:         gencore.burnToMint(collectionTokenMintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
288:         collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
289:     }
```

## [G-08] Add address(0) checks to prevent unnecessary external calls to primary addresses that are address(0)

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L429C1-L438C74)

**Function execution cost: 200 gas (CALL opcode = 100 gas * 2 for 2 empty primaryAddreses) - 15 gas (GT opcode = 3 gas * 5 since 3 addresses of artist and 2 addresses of team are checked for address(0)) = 185 gas saved (per call)**

It is possible for primary addresses to hold address(0) value since in the function [proposePrimaryAddressesAndPercentages()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L380), the artist can have only 1 royalty primary address set with max percentage of royalties the artist can receive. This means `primaryAdd1` and `primaryAdd2` are set to 0 with 0%. 

Now when paying the artist, there are unnecessary external call being made to the zero address on Line 450 and 451. This would cost 200 gas more (since CALL opcode costs 100 gas each). 

Since address(0) would receive 0 royalties, we can wrap each external call with a check that ensures that artistRoyalties[1/2/3] is greater than 0. If not, then we continue the rest of the calls which have royalties > 0.
```solidity
File: MinterContract.sol
443:         artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
444:         artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
445:         artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
446:         teamRoyalties1 = royalties * _teamperc1 / 100;
447:         teamRoyalties2 = royalties * _teamperc2 / 100;
449:         (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
450:         (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
451:         (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
452:         (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
453:         (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```

## [G-09] Simplify mathematical equation for `decreaserate` in Exponential Decay function (sales model 2) to save gas

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L551)

**Before VS After**

**Deployment cost: 5454331 - 5450662 = 3669 gas saved**

**Simplification of decreaserate equation:**

![](https://user-images.githubusercontent.com/109625274/282045110-1a77f583-43bb-4ed3-bc4f-834cf12ea846.png)

Instead of this:
```solidity
File: MinterContract.sol
569:  decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
```
Use this:
```solidity
File: MinterContract.sol
569:  decreaserate = (collectionPhases[_collectionId].collectionMintCost / ((tDiff + 1)*(tDiff + 2))) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
```

## [G-10] Return early when token reaches resting price in Linear Descending Sale (sales model 2) to save gas

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L552C3-L563C14)

**Before VS After**

**Deployment cost: 5454331 - 5449796 = 4535 gas saved**

**Function execution cost: 115 gas saved (per call)**

The below else block is to calculate the Linear Descending Sale price. Returning early (on Line 576 itself) when the token reaches the resting price in this model will save 115 gas since it would prevent the unnecessary check on Line 579, which will always be false when the token reaches resting price. 

Although the hardhat gas reporter did not display the function execution gas saved for the getPrice() function, I've tried to gauge the approximate gas that will be saved per function call.

On Line 576:
 - MSTORE to price memory variable (3 gas)

On Line 579:
 - 2 MLOADS for accessing `price` and `decreaserate` memory variables (2 * 3 MLOAD opcode cost = 6 gas)
 - `price - decreaserate` subtraction SUB operation (3 gas)
 - Greater than comparison (3 gas)
 - SLOAD to retrieve `collectionEndMintCost` (100 gas)

**Total Gas saved: 3 + 6 + 3 + 3 + 100 = 115 gas per call**

Instead of this:
```solidity
File: MinterContract.sol
572:             } else {
573:                 if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
574:                     price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
575:                 } else {
576:                     price = collectionPhases[_collectionId].collectionEndMintCost; 
577:                 }
578:             }
579:             if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
580:                 return price - decreaserate; 
581:             } else {
582:                 return collectionPhases[_collectionId].collectionEndMintCost;
583:             }
```
Use this (change on **Line 576**):
```solidity
File: MinterContract.sol
572:             } else {
573:                 if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
574:                     price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
575:                 } else {
576:                     return collectionPhases[_collectionId].collectionEndMintCost; 
577:                 }
578:             }
579:             if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
580:                 return price - decreaserate; 
581:             } else {
582:                 return collectionPhases[_collectionId].collectionEndMintCost;
583:             }
```

## [G-11] No need to track variable `highBid` in for loop of function `returnHighestBid()` to save gas

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L69C1-L74C14)

**Function execution cost: 3 gas * auctionInfoData[_tokenid].length + 100 * auctionInfoData[_tokenid].length (since MSTORE costs 3 gas and SLOAD costs 100 gas)**

Similar to the for loop in function [returnHighestBidder()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87), the for loop in function [returnHighestBid()](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L69C1-L74C14) does not need to track `highBid` on each iteration since the index holding the last non-zero bidder in the array is being tracked, which can be finally returned in the upcoming if check. **Note: The last non-zero bidder because participateToAuction allows users to participate only if msg.value is greater than previous bid. The non-zero because the last bidder in the array can cancel his bid.)

Instead of this:
```solidity
File: AuctionDemo.sol
70:             for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) { 
71:                 if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
72:                     highBid = auctionInfoData[_tokenid][i].bid; 
73:                     index = i;
74:                 }
75:             }

```
Use this:
```solidity
File: AuctionDemo.sol
70:             for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) { 
71:                 if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
72:                     index = i;
73:                 }
74:             }
```

## [G-12] Remove redundant check from function `returnHighestBidder()` to save gas

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L95)

Remove check on Line 95 since it is already checked in the if check in the for loop on Line 91.
```solidity
File: AuctionDemo.sol
90:         for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
91:             if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
92:                 index = i;
93:             }
94:         }
95:         if (auctionInfoData[_tokenid][index].status == true) {
96:                 return auctionInfoData[_tokenid][index].bidder;
97:             } else {
98:                 revert("No Active Bidder");
99:         }
```

## [G-13] Use `msg.sender` instead of accessing bidder from storage in function `cancelBid()` to save gas

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L128)

**Function execution cost: 98 gas saved (per call) (CALLER opcode costs 2 gas while SLOAD opcode costs 100 gas)**

We do not need to worry about the bidder not being the msg.sender since that is already checked in the function cancelBid() [here on Line 126](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L126).

Instead of this:
```solidity
File: AuctionDemo.sol
128:         (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
```
Use this:
```solidity
File: AuctionDemo.sol
128:         (bool success, ) = payable(msg.sender).call{value: auctionInfoData[_tokenid][index].bid}("");
``` 
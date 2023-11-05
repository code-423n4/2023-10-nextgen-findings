###  [G-01] Put common assignments outside of the if conditions.
```maxCollectionPurchases``` and ```setFinalSupplyTimeAfterMint``` are set in all branches. Put them outside of the if conditions. Besides, ```_collectionID * 10000000000``` are used twice in L155 and L156, cache its result for gas efficiency. (Caching the mapping items ```collectionAdditionalData[_collectionID]``` already revealed in bot report, leave it alone.)
```solidity
File: smart-contracts/NextGenCore.sol

147:    function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
148:        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
149:        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
150:            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
151:            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
152:            collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
153:            collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
154:            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
155:            collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
156:            collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
157:            wereDataAdded[_collectionID] = true;
158:        } else if (artistSigned[_collectionID] == false) {
159:            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
160:            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
161:            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
162:        } else {
163:            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
164:            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
165:        }
166:    }
```
[[L147-L166]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147-L166)

### [G-02] Optimize if statement conditions.
Change the if-statement structure helps to reduce the number of conditional judgements (and thus reduce gas) without change its result. In this case, the "else if" and "else" branches are optimized.
```solidity
File: smart-contracts/NextGenCore.sol

// origin:
345:        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
346:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
347:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
348:        } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
349:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
350:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
351:        }
352:        else {
353:            string memory b64 = ...;
354:            string memory _uri = ...;
355:            return _uri;
356:        }

// optimize:
345:        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false) {
346:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
347:            if (tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
348:                return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
349:            }
350:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
351:        }
352:        string memory b64 = ...;
353:        string memory _uri = ...;
354:        return _uri;
```
[[L345-L356]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L345-L356)

### [G-03] Unnecessary tokenId to collectionId mapping.
Collection id can be calculated by ```tokenId / 1e10```, which is more efficient than using a mapping storage.
```solidity
File: smart-contracts/NextGenCore.sol
67:    // maps tokends ids with collectionsids
68:    mapping (uint256 => uint256) private tokenIdsToCollectionIds;
```
[[L67-L68]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L67-L68)

### [G-04] Use the default value of ```timeOfLastMint``` under certain condition.
In below instances, ```timeOfLastMint``` is calulated and used only to check if one ```timePeriod``` has passed. Under condition ```lastMinData[col] == 0```, we actually can use the default value (0) of ```timeOfLastMint```, which has the same effect for the later time period comparion.
```solidity
File: smart-contracts/MinterContract.sol
241:            uint timeOfLastMint;
242:            if (lastMintDate[col] == 0) {
243:                // for public sale set the allowlist the same time as publicsale
244:                timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;
245:            } else {
246:                timeOfLastMint =  lastMintDate[col];
247:            }
248:            // uint calculates if period has passed in order to allow minting
249:            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
250:            // users are able to mint after a day passes
251:            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");

File: smart-contracts/MinterContract.sol
283:        uint timeOfLastMint;
284:        // check 1 per period
285:        if (lastMintDate[_collectionID] == 0) {
286:        // for public sale set the allowlist the same time as publicsale
287:            timeOfLastMint = collectionPhases[_collectionID].allowlistStartTime - collectionPhases[_collectionID].timePeriod;
288:        } else {
289:            timeOfLastMint =  lastMintDate[_collectionID];
290:        }
291:        // uint calculates if period has passed in order to allow minting
292:        uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
293:        // users are able to mint after a day passes
294:        require(tDiff>=1, "1 mint/period");
```
[[L241-L251]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L241-L251) | [[L283-L294]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L283-L294)

### [G-05] Unnecessary division operation.
For time period comparison, no need to divide by ```timePeriod``` first and then compare. Use ```block.timestamp - timeofLastMint >= collectionPhases[col].timePeriod``` instead, saving the gas of division operation.
```solidity
File: smart-contracts/MinterContract.sol
248:            // uint calculates if period has passed in order to allow minting
249:            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
250:            // users are able to mint after a day passes
251:            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");

File: smart-contracts/MinterContract.sol
291:        // uint calculates if period has passed in order to allow minting
292:        uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
293:        // users are able to mint after a day passes
294:        require(tDiff>=1, "1 mint/period");
```
[[L248-L251]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L248-L251) | [[L291-L294]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L291-L294)

### [G-06] Redundant calculation of ```collectionTokenMintIndex```.
```mintIndex``` is identical to ```collectionTokenMintIndex```, no need to calculate it again, just use ```collectionTokenMintIndex``` instead.
```solidity
File: smart-contracts/MinterContract.sol
263:        uint256 collectionTokenMintIndex;
264:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
265:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
266:        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
267:        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);

File: smart-contracts/MinterContract.sol
278:        uint256 collectionTokenMintIndex;
279:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
280:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
281:        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```
[[L263-L267]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L263-L267) | [[L278-L281]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L278-L281)

### [G-07] Consider use ```block.timestamp``` to update ```lastMintDate```.
Use ```block.timestamp``` to update ```lastMintDate``` for simplicity and efficiency.
```solidity
File: smart-contracts/MinterContract.sol
252:            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));

File: smart-contracts/MinterContract.sol
295:        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
```
[[L252]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L252) | [[L295]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L295)

###  [G-01] Switch common statements outside of the if conditions.
"maxCollectionPurchases" and "setFinalSupplyTimeAfterMint" are set in all branches. Switch them outside of the if conditions. Besides, "_collectionID * 10000000000" are used twice in L155 and L156, cache its result for gas efficiency. (Caching the mapping items "collectionAdditionalData[_collectionID]" already revealed in bot report, leave it alone.)
```solidity
File: smart-contract/NextGenCore.sol

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
File: smart-contract/NextGenCore.sol

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
Collection id can be calculated by "tokenId / 1e10", which is more efficient than using a mapping storage.
```solidity
File: smart-contracts/NextGenCore.sol
67:    // maps tokends ids with collectionsids
68:    mapping (uint256 => uint256) private tokenIdsToCollectionIds;
```
[[L67-L68]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L67-L68)

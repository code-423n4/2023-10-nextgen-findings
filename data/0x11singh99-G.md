## Gas Optimizations

| Number                                                                 | Issue                                                       | Instances |
| ---------------------------------------------------------------------- | :---------------------------------------------------------- | :-------: |
| [[G-01](#g-01-structs-can-be-packed-into-fewer-storage-slots)]         | `Structs` can be `packed` into fewer storage slots          |    10     |
| [[G-02](#g-02-cache-mappingarray-rather-then-re-reading-from-storage)] | Cache `Mapping`/`Array` rather then re-reading from storage |     3     |
| [[G-03](#g-03-use-already-cached-value)]                               | Use already `cached` value                                  |     3     |
| [[G-04](#g-04-dont-update-mappingarray-if-value-hasnt-change)]         | Don't update `mapping`/`array` if value hasn't change       |     1     |
| [[G-05](#g-05-check-_tokenid-for-zero-before-transfer)]                | Check `_tokenid` for `zero` before transfer                 |     1     |
| [[G-06](#g-06-use-calldata-instead-of-memory)]                         | Use `calldata` instead of `memory`                          |     7     |
| [[G-07](#g-07-can-make-the-variable-outside-of-the-loop)]              | Can make the variable outside of the loop                   |     2     |
| [[G-08](#g-08-use-ternary-instead-of-ifelse)]                          | Use `ternary` instead of `if()`/`else`                      |     1     |
| [[G-09](#g-09-remove-unnecessary-else-block)]                          | Remove unnecessary `else` block                             |     2     |

## [G-01] `Structs` can be `packed` into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

**Note: Missed by bot**

_10 Instances in 1 File_

### `artistPercentage` and `teamPercentage` can be reduced to `uint16` each and packed into single storage slot by reducing their size save total 2 SLOT in 2 structs.

According to [setPrimaryAndSecondarySplits()](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L369C5-L377C1) function's require statements we can see that `_artistPrSplit + _teamPrSplit` is equal to 100 and these both are assigned to artistPercentage and teamPercentage of the struct. so their value will always be 0 to 100 only. so **uint16** is enough to hold them. So that they can be packed and storage slots will be saved.

```solidity
File : smart-contracts/MinterContract.sol

63: struct royaltiesPrimarySplits {
64:      uint256 artistPercentage;
65:      uint256 teamPercentage;
66:  }


88: struct royaltiesSecondarySplits {
89:      uint256 artistPercentage;
90:      uint256 teamPercentage;
91:   }

```

[63-66](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L63C5-L66C6), [88-91](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L88C5-L91C6)

```diff
File : smart-contracts/MinterContract.sol

63: struct royaltiesPrimarySplits {
-64:      uint256 artistPercentage;
-65:      uint256 teamPercentage;
+64:      uint16 artistPercentage;
+65:      uint16 teamPercentage;
66:  }


88: struct royaltiesSecondarySplits {
-89:      uint256 artistPercentage;
-90:      uint256 teamPercentage;
+89:      uint16 artistPercentage;
+90:      uint16 teamPercentage;
91:   }

```

### `add1Percentage`, `add2Percentage` and `add3Percentage` can be reduced to `uin16` each and packed with `address` by reducing their size to save 6 more SLOT in 2 structs.

According to [proposePrimaryAddressesAndPercentages()](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L380C2-L390C6) and [proposeSecondaryAddressesAndPercentages()](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L394C5-L404C6) functions require statements we can see that `_add1Percentage + _add2Percentage + _add3Percentage` total will be equal to `artistPercentage`. And we can see in above instance `artistPercentage` value can be 0 to 100. So `_add1Percentage` , `_add2Percentage` , `_add3Percentage` value can be 0 to 100 only. And these values will be assigned to `add1Percentage` , `add2Percentage` , `add3Percentage` in `collectionPrimaryAddresses` and `collectionSecondaryAddresses` structs. So `add1Percentage` , `add2Percentage` , `add3Percentage` sizes can be reduced to `uint16` easily and it can save 3 SLOTs each in both structs.

**In bot report 2 SLOT already saved by packing boolean type status with address primaryAdd3 and secondaryAdd3 respectively in both structs. We save 6 more slots.**

```solidity
File : smart-contracts/MinterContract.sol

73: struct collectionPrimaryAddresses {
74:      address primaryAdd1;
75:      address primaryAdd2;
76:      address primaryAdd3;
77:      uint256 add1Percentage;
78:      uint256 add2Percentage;
79:      uint256 add3Percentage;
80:      bool status;
81:   }


98: struct collectionSecondaryAddresses {
99:      address secondaryAdd1;
100:     address secondaryAdd2;
101:     address secondaryAdd3;
102:     uint256 add1Percentage;
103:     uint256 add2Percentage;
104:     uint256 add3Percentage;
105:     bool status;
106:    }

```

[73-81](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L73C5-L81C6),
[98-106](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L98C5-L106C6)

```diff
File : smart-contracts/MinterContract.sol

73: struct collectionPrimaryAddresses {
74:      address primaryAdd1;
75:      address primaryAdd2;
76:      address primaryAdd3;
+77:      uint16 add1Percentage;
+78:      uint16 add2Percentage;
+79:      uint16 add3Percentage;
+80:      bool status;
-77:      uint256 add1Percentage;
-78:      uint256 add2Percentage;
-79:      uint256 add3Percentage;
-80:      bool status;
81:   }


98: struct collectionSecondaryAddresses {
99:      address secondaryAdd1;
100:     address secondaryAdd2;
101:     address secondaryAdd3;
+102:     uint16 add1Percentage;
+103:     uint16 add2Percentage;
+104:     uint16 add3Percentage;
+105:     bool status;
-102:     uint256 add1Percentage;
-103:     uint256 add2Percentage;
-104:     uint256 add3Percentage;
-105:     bool status;
106:    }

```

## [G-02] Cache `Mapping`/`Array` rather then re-reading from storage

**Note: These instances missed by bot-report**

_3 Instances in 1 File_

<details>
<summary>see instances</summary>

```solidity
File: smart-contracts/NextGenCore.sol

178:     function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
179:        require(msg.sender == minterContract, "Caller is not the Minter Contract");
180:        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
181:        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
182:            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
183:            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
184:        }
185:    }

```

[178-185](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178C1-L185C6)

```diff
File: smart-contracts/NextGenCore.sol

    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
+       uint256  _collection = collectionAdditionalData[_collectionID].collectionCirculationSupply +1;
-        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_collectionID].collectionCirculationSupply = _collection;
-        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
+        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= _collection) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }

```

```solidity
File: smart-contracts/NextGenCore.sol

189:     function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {
190:         require(msg.sender == minterContract, "Caller is not the Minter Contract");
191:         collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
192:         if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
193:            _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
194:            if (phase == 1) {
195:                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
196:         } else {
197:                tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
198:            }
199:        }
200:    }

```

[189-200](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L189C1-L200C6)

```diff
File: smart-contracts/NextGenCore.sol

    function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
+       uint256 _collection = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
-        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_collectionID].collectionCirculationSupply = _collection;
-        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
+        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= _collection) {
            _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
            if (phase == 1) {
                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
            } else {
                tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
            }
        }
    }

```

```solidity
File: smart-contracts/NextGenCore.sol

213:    function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
214:        require(msg.sender == minterContract, "Caller is not the Minter Contract");
215:        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
216:        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
217:        if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
218:            _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
219:            // burn token
220:            _burn(_tokenId);
221:            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
222:        }
223:    }

```

[213-223](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L213C1-L223C6)

```diff
File: smart-contracts/NextGenCore.sol

    function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
+       uint256 _collection = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
-        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = _collection;
-        if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
+        if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= _collection) {
            _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
            // burn token
            _burn(_tokenId);
            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
        }
    }

```

</details>

## [G-03] Use already `cached` value

_3 Instances in 1 File_

<details>
<summary>see instances</summary>

```solidity
File: smart-contracts/MinterContract.sol

263:        uint256 collectionTokenMintIndex;
264:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
265:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
266:        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
267:        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
268:        // burn and mint token
269:        address burner = msg.sender;
270:        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
271:        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
272:    }

```

[263-272](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L263C1-L272C6)

```diff
File: smart-contracts/MinterContract.sol

        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
-        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        // burn and mint token
        address burner = msg.sender;
-        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
+        gencore.burnToMint(collectionTokenMintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
    }

```

```solidity
File: smart-contracts/MinterContract.sol

278:        uint256 collectionTokenMintIndex;
279:        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
280:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
281:        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
282:        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
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
295:        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
296:        mintToAuctionData[mintIndex] = _auctionEndTime;
297:        mintToAuctionStatus[mintIndex] = true;
298:    }

```

[278-298](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L278C1-L298C6)

```diff
File: smart-contracts/MinterContract.sol

        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
-       uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
-       gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
+       gencore.airDropTokens(collectionTokenMintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
        uint timeOfLastMint;
        // check 1 per period
        if (lastMintDate[_collectionID] == 0) {
        // for public sale set the allowlist the same time as publicsale
            timeOfLastMint = collectionPhases[_collectionID].allowlistStartTime - collectionPhases[_collectionID].timePeriod;
        } else {
            timeOfLastMint =  lastMintDate[_collectionID];
        }
        // uint calculates if period has passed in order to allow minting
        uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
        // users are able to mint after a day passes
        require(tDiff>=1, "1 mint/period");
        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
-        mintToAuctionData[mintIndex] = _auctionEndTime;
-        mintToAuctionStatus[mintIndex] = true;
+        mintToAuctionData[collectionTokenMintIndex] = _auctionEndTime;
+        mintToAuctionStatus[collectionTokenMintIndex] = true;
    }

```

```solidity
File: smart-contracts/MinterContract.sol

358:        uint256 collectionTokenMintIndex;
359:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
360:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
361:        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
362:        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
363:        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
364:        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
365:    }

```

[358-365](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L358C1-L365C6)

```diff
File: smart-contracts/MinterContract.sol

        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
-       uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
-       gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
+       gencore.mint(collectionTokenMintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
    }

```

</details>

## [G-04] Don't update `mapping`/`array` if value hasn't change

Here `collectionArtistSecondaryAddresses[_collectionID].status` has not change.

_1 Instance in 1 File_

```solidity
File:smart-contracts/MinterContract.sol

395:        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
396:        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
397:        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1 = _secondaryAdd1;
398:        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2 = _secondaryAdd2;
399:        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3 = _secondaryAdd3;
400:        collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
401:        collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
402:        collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
403:        collectionArtistSecondaryAddresses[_collectionID].status = false;//@audit don't update

```

[395-403](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L395C1-L403C74)

## [G-05] Check `_tokenid` for `zero` before transfer

_1 Instance in 1 File_

```solidity
File: smart-contracts/AuctionDemo.sol

112:   IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);

```

[112](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112C17-L112C90)

## [G-06] Use `calldata` instead of `memory`

_7 Instances in 2 Files_

<details>
<summary>see instances</summary>

```solidity
File: smart-contracts/NextGenCore.sol

189:    function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {

238:    function updateCollectionInfo(uint256 _collectionID, string memory _newCollectionName, string memory _newCollectionArtist, string memory _newCollectionDescription, string memory _newCollectionWebsite, string memory _newCollectionLicense, string memory _newCollectionBaseURI, string memory _newCollectionLibrary, uint256 _index, string[] memory _newCollectionScript) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {

257:    function artistSignature(uint256 _collectionID, string memory _signature) public {

```

[189](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L189C4-L189C175), [238](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L238C4-L238C454), [257](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L257C5-L257C87)

```solidity
File: smart-contracts/MinterContract.sol

181:     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {

196:     function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {

276:     function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {

326:     function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {

```

[181](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L181C1-L181C231), [196](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196C5-L196C221), [276](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276C3-L276C200), [326](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326C5-L326C232)

</details>

## [G-07] Can make the variable outside of the loop

_2 Instances in 1 File_

<details>
<summary>see instances</summary>

```solidity
File: smart-contracts/MinterContract.sol

183:        uint256 collectionTokenMintIndex;
184:        for (uint256 y=0; y< _recipients.length; y++) {
185:            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
186:            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
187:            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
188:                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
189:                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
190:            }
191:        }

```

[183-191](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L183C1-L191C10)

```diff
File: smart-contracts/MinterContract.sol

        uint256 collectionTokenMintIndex;
+       uint256 mintIndex;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
-                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
+                mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
            }
        }

```

```solidity
File: smart-contracts/MinterContract.sol

        for(uint256 i = 0; i < _numberOfTokens; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);

```

[234-235](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L234C1-L235C94)

```diff
File: smart-contracts/MinterContract.sol

+       uint256 mintIndex;
        for(uint256 i = 0; i < _numberOfTokens; i++) {
-            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
+            mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);

```

</details>

## [G-08] Use `ternary` instead of `if()`/`else`

**Note: This instance missed by bot-report**

_1 Instance in 1 File_

```solidity
File: smart-contracts/XRandoms.sol

28:        if (id==0) {
29:            return wordsList[id];
30:        } else {
31:            return wordsList[id - 1];
32:        }

```

[28-32](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28C1-L32C10)

## [G-09] Remove unnecessary `else` block

_2 Instances in 1 File_

```solidity
File : smart-contracts/AuctionDemo.sol

118:  } else {}

141:  } else {}

```

[188](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L118), [141](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L141)

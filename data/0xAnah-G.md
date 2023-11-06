# NEXTGEN GAS OPTIMISATIONS


## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."








## [G-01] Refactor `NextGenRandomizerNXT.calculateTokenHash()` function to avoid 1 `SLOAD` (Gcoldaccess) `2100` gas units.
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L58

The require statement `require(msg.sender == gencore)` of the `NextGenRandomizerNXT.calculateTokenHash()` ensures that `msg.sender` equals `gencore` else the function reverts. Since we are sure that for the function to proceed the value of `msg.sender` must equal `gencore` but we also know that the value of `gencoreContract` is always `INextGenCore(gencore)` so rather than having to read `gencoreContract` from state which would cost `2100` gas units we could use `INextGenCore(msg.sender)` since `msg.sender` equals `gencore` and is more cheaper. The diff below shows how we could refactor the function.

```solidity
file: smart-contracts/RandomizerNXT.sol

55:    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
56:        require(msg.sender == gencore);
57:        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
58:        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
59:    }
```

```diff
diff --git a/smart-contracts/RandomizerNXT.sol b/smart-contracts/RandomizerNXT.sol
index 0c7818c..caf8cf7 100644
--- a/smart-contracts/RandomizerNXT.sol
+++ b/smart-contracts/RandomizerNXT.sol
@@ -55,7 +55,7 @@ contract NextGenRandomizerNXT {
     function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
         require(msg.sender == gencore);
         bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
-        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
+        INextGenCore(msg.sender).setTokenHash(_collectionID, _mintIndex, hash);
     }
```
```
Estimated gas saved: 2097 gas units
```




## [G-02]  Move lesser gas costing require checks to the top
Require() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 1 Instance
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L327-#L329

The require checks of the `burnOrSwapExternalToMint()` function can be re-ordered to make the function better gas efficient. The `require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs")` is cheaper (as it involves just reading the `setMintingCosts[_mintCollectionID]` mapping) compared to `require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn")` (which depends on the computation of `externalCol`). The cheaper require statement should be moved to the top of the function so that in scenarios in which it would fail the EVM would not have to perform gas consuming operations of having to compute the value of `externalCol` and having to read two mappings from state. The diff below shows how the code could be refactored:

```solidity
file: smart-contracts/MinterContract.sol

326:    function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
327:        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
328:        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
329:        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
330:        address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
.
.
.
365:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..0635afd 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -324,9 +324,9 @@ contract NextGenMinterContract is Ownable {
     // burn or swap to mint (requires contract approval)

     function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
+        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
         bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
         require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
-        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
         address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
         if (msg.sender != ownerOfToken) {
             bool isAllowedToMint;
```




## [G-03] Refactor the following Functions to avoid setting storage when the value hasn't changed
#### Please note these instances are different from the ones in the bots report.

### 2 Instances

1. #### Refactor `proposePrimaryAddressesAndPercentages()` to avoid re-setting the value of  `collectionArtistPrimaryAddresses[_collectionID].status` since the value has not changed.
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L380-#L390

In the `NextGenMinterContract.proposePrimaryAddressesAndPercentages()` the require statement `require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved")` ensures that the value of `collectionArtistPrimaryAddresses[_collectionID].status` is `false` else the function would revert. Since the require statement ensures that the value of `collectionArtistPrimaryAddresses[_collectionID].status` is `false` it is absolutely unnecessary to still set the value of `collectionArtistPrimaryAddresses[_collectionID].status` to `false` later in the function. The statement setting the value of `collectionArtistPrimaryAddresses[_collectionID].status` to `false` should be removed from the function in doing this we would avoid `SSTORE` `20000` gas units if it is the first time the storage slot is being set or `SSTORE` `2900` for subsequents storage set.

```solidity
file: smart-contracts/MinterContract.sol

380:    function proposePrimaryAddressesAndPercentages(uint256 _collectionID, address _primaryAdd1, address _primaryAdd2, address _primaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
381:        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
382:        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");
383:        collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
384:        collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
385:        collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
386:        collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
387:        collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
388:        collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
389:        collectionArtistPrimaryAddresses[_collectionID].status = false;
390:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..1baa91e 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -386,7 +386,7 @@ contract NextGenMinterContract is Ownable {
         collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
         collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
         collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
-        collectionArtistPrimaryAddresses[_collectionID].status = false;
+
     }
```
```
Estimated gas saved: 20000 gas units
```


2. #### Refactor `proposeSecondaryAddressesAndPercentages()` to avoid re-setting the value of  `collectionArtistSecondaryAddresses[_collectionID].status` since the value has not changed.
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L394-#L404

In the `NextGenMinterContract.proposeSecondaryAddressesAndPercentages()` the require statement `require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved")` ensures that the value of `collectionArtistSecondaryAddresses[_collectionID].status` is `false` else the function would revert. Since the require statement ensures that the value of `collectionArtistSecondaryAddresses[_collectionID].status` is `false` it is absolutely unnecessary to still set the value of `collectionArtistSecondaryAddresses[_collectionID].status` to `false` later in the function. The statement setting the value of `collectionArtistSecondaryAddresses[_collectionID].status` to `false` should be removed from the function in doing this we would avoid `SSTORE` `20000` gas units if it is the first time the storage slot is being set or `SSTORE` `2900` for subsequents storage set.

```solidity
file: smart-contracts/MinterContract.sol

394:    function proposeSecondaryAddressesAndPercentages(uint256 _collectionID, address _secondaryAdd1, address _secondaryAdd2, address _secondaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposeSecondaryAddressesAndPercentages.selector) {
395:        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
396:        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
397:        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1 = _secondaryAdd1;
398:        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2 = _secondaryAdd2;
399:        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3 = _secondaryAdd3;
400:        collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
401:        collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
402:        collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
403:        collectionArtistSecondaryAddresses[_collectionID].status = false;
404:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..98c2079 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -400,7 +400,7 @@ contract NextGenMinterContract is Ownable {
         collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
         collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
         collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
-        collectionArtistSecondaryAddresses[_collectionID].status = false;
+
     }

     // function to accept primary addresses and percentages
```
```
Estimated gas saved: 20000 gas units
```




## [G-04] Refactor the following functions to reduce number of external calls made

### 5 Instances
1. #### Refactor `NextGenMinterContract.airDropTokens()` to reduce number of external calls made
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L181-#L192

In the `NextGenMinterContract.airDropTokens()` function external calls such as `gencore.viewTokensIndexMin(_collectionID)` and `gencore.viewCirSupply(_collectionID)` were performed multiple times in the function in both the outer and nested for loops this would incur `100` gas units for the external call plus cost of executing the function for every of the external call per loop iteration. We could perform this external calls and cache the results before the first (or outer) loop since the iterations does not affect the result of the external calls. The diff below shows how the code could be refactored:

```solidity
file: smart-contracts/MinterContract.sol

181:    function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
182:        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
183:        uint256 collectionTokenMintIndex;
184:        for (uint256 y=0; y< _recipients.length; y++) {
185:            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
186:            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
187:            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
188:                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
189:                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
190:            }
191:        }
192:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..3794e66 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -181,11 +181,11 @@ contract NextGenMinterContract is Ownable {
     function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
         uint256 collectionTokenMintIndex;
+        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
         for (uint256 y=0; y< _recipients.length; y++) {
-            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
+            collectionTokenMintIndex = mintIndex + _numberOfTokens[y] - 1;
             require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
             for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
-                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                 gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
             }
         }
```


2. #### Refactor `NextGenMinterContract.mint()` to reduce number of external calls made
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196-#L254

In the `NextGenMinterContract.mint()` function the external calls `gencore.viewCirSupply(col)` and `gencore.viewTokensIndexMin(col)` were performed multiple times in the function even inside the loop even though the loops iteration does not affect the result returned by these external calls.  For every instance these external calls were made would incur a cost of  `100` gas unit for the external call plus cost of executing the function. We can reduce the gas cost of the `NextGenMinterContract.mint()` function if perform this external calls once, cache their returned values and use these values subsequently rather than having to perform the external calls again. The diff below shows how the code could be refactored:
```solidity
file: smart-contracts/MinterContract.sol

196:    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
.
.
.
230:        uint256 collectionTokenMintIndex;
231:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
232:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
233:        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
234:        for(uint256 i = 0; i < _numberOfTokens; i++) {
235:            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
236:            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
237:        }
238:        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
239:        // control mechanism for sale option 3
240:        if (collectionPhases[col].salesOption == 3) {
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
252:            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
253:        }
254:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..bcb2398 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -228,11 +228,12 @@ contract NextGenMinterContract is Ownable {
             revert("No minting");
         }
         uint256 collectionTokenMintIndex;
-        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
+        uint256 collectionCirSupply = gencore.viewCirSupply(col);
+        uint256 mintIndex = collectionCirSupply + gencore.viewTokensIndexMin(col)
+        collectionTokenMintIndex = mintIndex + _numberOfTokens - 1;
         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
         require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
         for(uint256 i = 0; i < _numberOfTokens; i++) {
-            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
             gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
         }
         collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
@@ -249,7 +250,7 @@ contract NextGenMinterContract is Ownable {
             uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
             // users are able to mint after a day passes
             require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
-            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
+            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (collectionCirSupply - 1));
         }
     }
```

3. #### Refactor `NextGenMinterContract.burnToMint()` to reduce number of externals call made
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L258-#L272

In the `NextGenMinterContract.burnToMint()` function the external calls `gencore.viewTokensIndexMin(_mintCollectionID)` and `gencore.viewCirSupply(_mintCollectionID)` were made multiple times. On [Line 264](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L264) the sum of the return values of these external calls were saved to variable `collectionTokenMintIndex` this was also repeated on [Line 267](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267) but this time saved to variable `mintIndex`. Having to repeat the operations of [Line 264](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L264) on [Line 267](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267) is absolutely unnecessary. The code should be refactored such that [Line 267](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267) is removed and `collectionTokenMintIndex` is used in place of `mintIndex` in the statement on [Line 270](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L270). The diff below shows how these could be done:

```solidity
file: smart-contracts/MinterContract.sol

258:    function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
259:        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
260:        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
261:        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
262:        // minting new token
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

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..83f3078 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -264,10 +264,9 @@ contract NextGenMinterContract is Ownable {
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


4. #### Refactor `NextGenMinterContract.mintAndAuction()` to reduce number of external calls made
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276-#L298

In the `NextGenMinterContract.mintAndAuction()` function the external calls `gencore.viewTokensIndexMin(_collectionID)` and `gencore.viewCirSupply(_collectionID)` were made multiple times. On [Line 279](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L279) the sum of the return values of these external calls were saved to variable `collectionTokenMintIndex` this was also repeated on [Line 281](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L281) but this time saved to variable `mintIndex`. Having to repeat the operations of [Line 279](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L279) on [Line 281](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L281) is absolutely unnecessary. The code should be refactored such that [Line 281](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L281) is removed and `collectionTokenMintIndex` is used in place of `mintIndex` subsequently. The diff below shows how the function should be refactored:

```solidity
file: smart-contracts/MinterContract.sol

276:    function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
277:        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
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

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..809e8d3 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -275,11 +275,11 @@ contract NextGenMinterContract is Ownable {

     function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
+        uint256 collectionCirSupply = gencore.viewCirSupply(_collectionID);
         uint256 collectionTokenMintIndex;
-        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
+        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + collectionCirSupply;
         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
-        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
-        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
+        gencore.airDropTokens(collectionTokenMintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
         uint timeOfLastMint;
         // check 1 per period
         if (lastMintDate[_collectionID] == 0) {
@@ -292,9 +292,9 @@ contract NextGenMinterContract is Ownable {
         uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
         // users are able to mint after a day passes
         require(tDiff>=1, "1 mint/period");
-        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
-        mintToAuctionData[mintIndex] = _auctionEndTime;
-        mintToAuctionStatus[mintIndex] = true;
+        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (collectionCirSupply - 1));
+        mintToAuctionData[collectionTokenMintIndex] = _auctionEndTime;
+        mintToAuctionStatus[collectionTokenMintIndex] = true;
     }
```


5. #### Refactor `NextGenMinterContract.burnOrSwapExternalToMint()` to reduce number of external calls made
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326-#L365

In the `NextGenMinterContract.burnOrSwapExternalToMint()` function the external calls `gencore.viewTokensIndexMin(col)` and `gencore.viewCirSupply(col)` were made multiple times. On [Line 359](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L359) the sum of the return values of these external calls were saved to variable `collectionTokenMintIndex` this was also repeated on [Line 362](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362) but this time saved to variable `mintIndex`. Having to repeat the operations of [Line 359](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L359) on [Line 362](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362) is absolutely unnecessary. The code should be refactored such that [Line 362](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362) is removed and `collectionTokenMintIndex` is used in place of `mintIndex` in the statement on [Line 363](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L363). The diff below shows how these could be done:

```solidity
file: smart-contracts/MinterContract.sol

326:    function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
.
.
.
358:        uint256 collectionTokenMintIndex;
359:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
360:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
361:        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
362:        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
363:        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
364:        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
365:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..977dc47 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -359,8 +359,7 @@ contract NextGenMinterContract is Ownable {
         collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
         require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
-        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
-        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
+        gencore.mint(collectionTokenMintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
         collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
     }
```





## [G-05] Refactor public / private function to avoid unnecessary SLOAD 
The private function below read storage slots that are previously read in the public function that invokes it. We can refactor both the public and private function such that the public function passes cached storage variables as arguments to the private function thereby avoiding the extra storage read (SLOAD) that would otherwise be incurred in the private function.

### 1 Instance
1. #### Refactor `tokenURI()` and `getTokenName()` such that `getTokenName()` does not have to read `tokenIdsToCollectionIds[tokenId]` value from state.
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L343-#L364

We can refactor the public function `tokenURI()` and the private function `getTokenName()` such that we cache the value of `tokenIdsToCollectionIds[tokenId]` and pass this value as an argument to the `getTokenName()` function. In doing this we would save on gas in the following ways: 
1) on [line 362](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L362) we would avoid an `SLOAD`(Gwarmaccess) `100` gas units and `KECCAK256` `30` gas units in reading the value of `tokenIdsToCollectionIds[tokenId]`.
2) on [line 363](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L362) we would avoid 2 `JUMP` `16` gas units in having to call the `viewColIDforTokenID()` function and an `SLOAD`(Gwarmaccess) `100` gas units and `KECCAK256` `30` gas units in reading the value of `tokenIdsToCollectionIds[tokenId]` in executing the `viewColIDforTokenID()` function. The diff below shows how the changes could be applied:

```solidity
file: smart-contracts/NextGenCore.sol

343:    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
.
.
.
353:            string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));
354:            string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));
355:            return _uri;
356:        }
357:    }

    // function get Name

361:    function getTokenName(uint256 tokenId) private view returns(string memory)  {
362:        uint256 tok = tokenId - collectionAdditionalData[tokenIdsToCollectionIds[tokenId]].reservedMinTokensIndex;
363:        return string(abi.encodePacked(collectionInfo[viewColIDforTokenID(tokenId)].collectionName, " #" ,tok.toString()));
364:    }
```

```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..7c22d15 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -342,25 +342,26 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {

     function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
         _requireMinted(tokenId);
-        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
-            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
+        uint256 colId = tokenIdsToCollectionIds[tokenId];
+        if (onchainMetadata[colId] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
+            string memory baseURI = collectionInfo[colId].collectionBaseURI;
             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
-        } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
-            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
+        } else if (onchainMetadata[colId] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
+            string memory baseURI = collectionInfo[colId].collectionBaseURI;
             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
         }
         else {
             string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));
-            string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));
+            string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId, colId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));
             return _uri;
         }
     }

     // function get Name

-    function getTokenName(uint256 tokenId) private view returns(string memory)  {
-        uint256 tok = tokenId - collectionAdditionalData[tokenIdsToCollectionIds[tokenId]].reservedMinTokensIndex;
-        return string(abi.encodePacked(collectionInfo[viewColIDforTokenID(tokenId)].collectionName, " #" ,tok.toString()));
+    function getTokenName(uint256 tokenId, uint256 colId) private view returns(string memory)  {
+        uint256 tok = tokenId - collectionAdditionalData[colId].reservedMinTokensIndex;
+        return string(abi.encodePacked(collectionInfo[colId].collectionName, " #" ,tok.toString()));
```




## [G-06] Unnecessary variable declaration

### 4 Instances
1. #### Declaration of `col` variable is unnecessary
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L341

The declaration of `col` variable on [line 341](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L341) is unnecessary because rather than having to cache the value of `_mintCollectionID` in `col` we can use the `_mintCollectionID` variable directly. In doing this we would avoid stack operation of having to copy value of `_mintCollectionID` to `col` thereby saving `3` gas units.

```solidity
file: smart-contracts/MinterContract.sol

326:    function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
.
.
.        
341:        uint256 col = _mintCollectionID;    //@audit declaration of col unnecessary
342:        address mintingAddress;
343:        uint256 phase;
344:        string memory tokData = _tokenData;
345:        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
346:            phase = 1;
347:            bytes32 node;
348:            node = keccak256(abi.encodePacked(_tokenId, tokData));
349:            mintingAddress = ownerOfToken;
350:            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');            
351:        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
352:            phase = 2;
353:            mintingAddress = ownerOfToken;
354:            tokData = '"public"';
355:        } else {
356:            revert("No minting");
357:        }
358:        uint256 collectionTokenMintIndex;
359:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
360:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
361:        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
362:        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
363:        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
364:        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
365:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..433204a 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -338,17 +338,16 @@ contract NextGenMinterContract is Ownable {
         }
         require(_tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1], "Token id does not match");
         IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
-        uint256 col = _mintCollectionID;
         address mintingAddress;
         uint256 phase;
         string memory tokData = _tokenData;
-        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
+        if (block.timestamp >= collectionPhases[_mintCollectionID].allowlistStartTime && block.timestamp <= collectionPhases[_mintCollectionID].allowlistEndTime) {
             phase = 1;
             bytes32 node;
             node = keccak256(abi.encodePacked(_tokenId, tokData));
             mintingAddress = ownerOfToken;
-            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');
-        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
+            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[_mintCollectionID].merkleRoot, node), 'invalid proof');
+        } else if (block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp <= collectionPhases[_mintCollectionID].publicEndTime) {
             phase = 2;
             mintingAddress = ownerOfToken;
             tokData = '"public"';
@@ -356,12 +355,12 @@ contract NextGenMinterContract is Ownable {
             revert("No minting");
         }
         uint256 collectionTokenMintIndex;
-        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
-        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
-        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
-        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
-        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
-        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
+        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
+        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
+        require(msg.value >= (getPrice(_mintCollectionID) * 1), "Wrong ETH");
+        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(_mintCollectionID);
+        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, _mintCollectionID, phase);
+        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
```
```
Estimated gas saved: 3 gas units
```


2. #### The declarations of variables `tm1`, `tm2` and `colId` are unnecessary
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L421
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L422
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L423

In the `payArtist` function below declarations of variables [tm1](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L421) , [tm2](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L422) and [colId](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L341) are unnecessary because they are used to cache function arguments `_team1`, `_team2` and `_collectionID` respectively. Rather than declaring this variables and caching the function arguments we can use these function arguments directly. In doing this we would avoid stack operation of having to copy value of the function arguments to these variables thereby thereby saving `3` gas units for each copy operation and `9` gas units in total.

```solidity
file: smart-contracts/MinterContract.sol

 415:   function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
.
.
.
421:        address tm1 = _team1;    //@audit declaration of col unnecessary
422:        address tm2 = _team2;    //@audit declaration of col unnecessary
423:        uint256 colId = _collectionID;    //@audit declaration of col unnecessary
424:        uint256 artistRoyalties1;
425:        uint256 artistRoyalties2;
426:        uint256 artistRoyalties3;
427:        uint256 teamRoyalties1;
428:        uint256 teamRoyalties2;
429:        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
430:        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
431:        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
432:        teamRoyalties1 = royalties * _teamperc1 / 100;
433:        teamRoyalties2 = royalties * _teamperc2 / 100;
434:        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
435:        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
436:        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
437:        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
438:        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
439:        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);
440:        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);
441:        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
442:        emit PayTeam(tm1, success4, teamRoyalties1);
443:        emit PayTeam(tm2, success5, teamRoyalties2);
444:    }
```

```diff
diff --git a/smart-contracts/MinterContract.sol b/smart-contracts/MinterContract.sol
index df50841..87e502f 100644
--- a/smart-contracts/MinterContract.sol
+++ b/smart-contracts/MinterContract.sol
@@ -426,21 +426,21 @@ contract NextGenMinterContract is Ownable {
         uint256 artistRoyalties3;
         uint256 teamRoyalties1;
         uint256 teamRoyalties2;
-        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
-        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
-        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
+        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[_collectionID].add1Percentage / 100;
+        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[_collectionID].add2Percentage / 100;
+        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[_collectionID].add3Percentage / 100;
         teamRoyalties1 = royalties * _teamperc1 / 100;
         teamRoyalties2 = royalties * _teamperc2 / 100;
-        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
-        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
-        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
-        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
-        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
-        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);
-        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);
-        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
-        emit PayTeam(tm1, success4, teamRoyalties1);
-        emit PayTeam(tm2, success5, teamRoyalties2);
+        (bool success1, ) = payable(collectionArtistPrimaryAddresses[_collectionID].primaryAdd1).call{value: artistRoyalties1}("");
+        (bool success2, ) = payable(collectionArtistPrimaryAddresses[_collectionID].primaryAdd2).call{value: artistRoyalties2}("");
+        (bool success3, ) = payable(collectionArtistPrimaryAddresses[_collectionID].primaryAdd3).call{value: artistRoyalties3}("");
+        (bool success4, ) = payable(_team1).call{value: teamRoyalties1}("");
+        (bool success5, ) = payable(_team2).call{value: teamRoyalties2}("");
+        emit PayArtist(collectionArtistPrimaryAddresses[_collectionID].primaryAdd1, success1, artistRoyalties1);
+        emit PayArtist(collectionArtistPrimaryAddresses[_collectionID].primaryAdd2, success2, artistRoyalties2);
+        emit PayArtist(collectionArtistPrimaryAddresses[_collectionID].primaryAdd3, success3, artistRoyalties3);
+        emit PayTeam(_team1, success4, teamRoyalties1);
+        emit PayTeam(_team2, success5, teamRoyalties2);
     }
```
```
Estimated gas saved: 9 gas uints
```





## [G-07] Cache the result of some computation so as to avoid re-computing them

### 1 Instance
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L345&&#L348

In the `tokenURI` function below we can cache the result of `onchainMetadata[tokenIdsToCollectionIds[tokenId]]` and `tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000` so that in scenarios where the if statement comdition results to a `false` the EVM would not need to re-compute the above computations. The diff below shows how the code could be refactored.

```solidity
file: smart-contracts/NextGenCore.sol

343:    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
344:        _requireMinted(tokenId);
345:        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
346:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
347:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
348:        } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
349:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
350:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
351:        }
352:        else {
353:            string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));
354:            string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));
355:            return _uri;
356:        }
357:    }
```

```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..08e78c3 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -342,10 +342,13 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
         _requireMinted(tokenId);
-        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
+        bool metaDataTokenIdsToColId = !onchainMetadata[tokenIdsToCollectionIds[tokenId]];
+        bool isHash = tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000;
+
+        if (metaDataTokenIdsToColId && isHash) {
             string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
-        } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
+        } else if (metaDataTokenIdsToColId && !isHash) {
             string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
             return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
         }
```




## [G-08] Structs can be modified to fit in fewer storage slots

### 1 Instance
1. #### The `setFinalSupplyTimeAfterMint` member of the `collectionAdditonalDataStructure` struct can be safely modified from type `uint` to `uint40`
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L44-#L54

We can safely modify the `setFinalSupplyTimeAfterMint` member of the `collectionAdditonalDataStructure` struct since its value represents a timestamp and `uint40` type is enough as 2^40 as a unix timestamp is ~35k years into the future which should be enough. In modifying `setFinalSupplyTimeAfterMint` to `uint40` we would reduce the number of storage slots required for the `collectionAdditonalDataStructure` struct from 9 to 8. In Implementing this change we would avoid an extra Gsset (20000 gas) for the first setting of the struct also subsequent reads and writes would have lesser gas cost.


```solidity
file: smart-contracts/NextGenCore.sol

44:    struct collectionAdditonalDataStructure {
45:        address collectionArtistAddress;         // (20 bytes)
46:        uint256 maxCollectionPurchases;          // (32 bytes)
47:        uint256 collectionCirculationSupply;     // (32 bytes)
48:        uint256 collectionTotalSupply;           // (32 bytes)
49:        uint256 reservedMinTokensIndex;          // (32 bytes)
50:        uint256 reservedMaxTokensIndex;          // (32 bytes)
51:        uint setFinalSupplyTimeAfterMint;        // (32 bytes)
52:        address randomizerContract;              // (20 bytes)
53:        IRandomizer randomizer;                  // (20 bytes)
54:    }
```

```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..8ea1c08 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -48,7 +48,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
         uint256 collectionTotalSupply;
         uint256 reservedMinTokensIndex;
         uint256 reservedMaxTokensIndex;
-        uint setFinalSupplyTimeAfterMint;
+        uint90 setFinalSupplyTimeAfterMint;
         address randomizerContract;
         IRandomizer randomizer;
     }
```
```
Estimated gas saved: 20000
```




## [G-09] Add unchecked blocks for additions where the operands cannot overflow
There are some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.
### Please note this instance was not included in the bots report.


### 1 Instance
1. #### Its impossible for the addition `newCollectionIndex = newCollectionIndex + 1` to overflow so it can be unchecked
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110

It is impossible for the addition `newCollectionIndex = newCollectionIndex + 1` to overflow because the initial value of `newCollectionIndex` is 0 and adding 1 to 0 cannot cause a `uint256` variable to overflow therefore we can bypass the compiler imputed checks by putting the addition in an unchecked block. Implemeting this would save about `30` gas units 

```solidity
file: smart-contracts/NextGenCore.sol

108:    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
109:        adminsContract = INextGenAdmins(_adminsContract);
110:        newCollectionIndex = newCollectionIndex + 1;
111:        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);
112:    }
```
```
Estimated gas saved: 30 gas units
```




## [G-10] Use += for mappings
Using += for mappings saves 40 gas due to not having to recalculate the mapping's value's hash.
### Please note this instance was not included in the bots report.

### Instances
- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L216

```solidity
file: smart-contracts/NextGenCore.sol

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

```diff
diff --git a/smart-contracts/NextGenCore.sol b/smart-contracts/NextGenCore.sol
index 6d294ed..d5e053e 100644
--- a/smart-contracts/NextGenCore.sol
+++ b/smart-contracts/NextGenCore.sol
@@ -213,7 +213,7 @@ contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
     function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
         require(msg.sender == minterContract, "Caller is not the Minter Contract");
         require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
-        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
+        collectionAdditionalData[_mintCollectionID].collectionCirculationSupply += 1;
         if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
             _mintProcessing(mintIndex, ownerOf(_tokenId), tokenData[_tokenId], _mintCollectionID, _saltfun_o);
             // burn token
```
```
Estimated gas saved: 40 gas units
```





## CONCLUSION
As you embark on the journey of incorporating the recommended changes, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.




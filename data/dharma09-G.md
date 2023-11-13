## **Table Of Contents**

- [Table Of Contents](#)
    - [Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it)](#)
    - [Can make the variable outside the loop to save gas](#)
    - [Use the already cached variable instead of reading from storage(Save 1 SLOAD: 100 Gas)](#)**
    - [Use assembly for back to back external call](#)
    - [`a++` is more gas-efficient than `a = a + 1` in Solidity](#)
    - [Reuse variables where possible to minimize storage operations](#)
    - [Some computations are safe to be included under an `unchecked` block to save gas](#)

### [G-01] **Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it)**

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L269

```solidity
File: smart-contracts/MinterContract.sol
258: function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
        // minting new token
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        // burn and mint token
        address burner = msg.sender; //@audit
        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
    }
```

```diff
File: smart-contracts/MinterContract.sol
258: function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
        // minting new token
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        // burn and mint token
-        address burner = msg.sender;
-        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
+        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, msg.sender);
        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
    }
```

## [G-02] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L188

```solidity
File: smart-contracts/MinterContract.sol
188: uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID); //@audit
```

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L235

```solidity
File: smart-contracts/MinterContract.sol
235: uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col); //@audit
```

### [G-03] **Use the already cached variable instead of reading from storage(Save 1 SLOAD: 100 Gas)**

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L301

```solidity
File: smart-contracts/NextGenCore.sol
299: function setTokenHash(uint256 _collectionID, uint256 _mintIndex, bytes32 _hash) external {
        require(msg.sender == collectionAdditionalData[_collectionID].randomizerContract);
        require(tokenToHash[_mintIndex] == 0x0000000000000000000000000000000000000000000000000000000000000000);
        tokenToHash[_mintIndex] = _hash; //@audit
    }
```

```diff
File: smart-contracts/NextGenCore.sol
299: function setTokenHash(uint256 _collectionID, uint256 _mintIndex, bytes32 _hash) external {
        require(msg.sender == collectionAdditionalData[_collectionID].randomizerContract);
-        require(tokenToHash[_mintIndex] == 0x0000000000000000000000000000000000000000000000000000000000000000);
+        require(_hash == 0x0000000000000000000000000000000000000000000000000000000000000000);
        tokenToHash[_mintIndex] = _hash; //@audit
    }
```

### [G-04] Use assembly for back to back external call

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L189

```solidity
File: smart-contracts/MinterContract.sol
189: gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID); //@audit back-to back external call
```

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362

```solidity

File: smart-contracts/MinterContract.sol
362: uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col); //@audit back to back external call
```

### [G-05] **`a++`** is more gas-efficient than **`a = a + 1`** in Solidity.

The **`++`** operator is a shorthand increment operator that is optimized at the EVM (Ethereum Virtual Machine) level and typically results in lower gas costs.

Using **`a++`** is a more concise way of expressing an increment operation, and it often compiles to more efficient bytecode. On the other hand, **`a = a + 1`** involves an additional assignment operation, which may result in slightly higher gas costs due to the extra bytecode generated.

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L197

```solidity
File: smart-contracts/NextGenCore.sol
140:               newCollectionIndex = newCollectionIndex + 1;

183: 		            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;

195: 		                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;

197: 		                tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;

208: 		        burnAmount[_collectionID] = burnAmount[_collectionID] + 1;

221: 		            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
```

### [G-06] Reuse variables where possible to minimize storage operations.

Combine variable assignments to reduce the number of instructions and potentially save gas.

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L267

```solidity
File: smart-contracts/MinterContract.sol
258: function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
        // minting new token
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID); //@audit
        // burn and mint token
        address burner = msg.sender;
        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
    }
```

```diff
258: function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");
        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");
        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
        // minting new token
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");
-        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID); //@audit
        // burn and mint token
        address burner = msg.sender;
-       gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
+       gencore.burnToMint(collectionTokenMintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);
        collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;
    }
```

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L281

```solidity
File: smart-contracts/MinterContract.sol
276: function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID); //@audit
        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
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
        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1)); //@audit
        mintToAuctionData[mintIndex] = _auctionEndTime;
        mintToAuctionStatus[mintIndex] = true;
    }
```

```diff
File: smart-contracts/MinterContract.sol
276: function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
-        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID); //@audit
+        uint256 mintIndex = collectionTokenMintIndex;
        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);
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
        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1)); //@audit
        mintToAuctionData[mintIndex] = _auctionEndTime;
        mintToAuctionStatus[mintIndex] = true;
    }
```

### [G-07] Some computations are safe to be included under an `unchecked` block to save gas

`newCollectionIndex = newCollectionIndex + 1` can use it as `newCollectionIndex` is `uint256`:

- https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L140

```solidity
File: smart-contracts/NextGenCore.sol
140: newCollectionIndex = newCollectionIndex + 1; //@audit use unchecked
```
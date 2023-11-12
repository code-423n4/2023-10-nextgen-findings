## Gas report



#### [G-1] - Dont need read variable, the value is known
If we find ourselves inside an if block, we know exactly what value the variable has. And delete else block.
```diff
function getWord(uint256 id) private pure returns (string memory) {
	...
-  if (id==0) {
-           return wordsList[id];
-       } else {
-           return wordsList[id - 1];
-       }
+  if(id==0) return wordsList[0]; // <-- wordsList[0] 
+  return wordsList[id - 1];
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L15-L33

### [G-2] Make variable numWords constant
The NumWords variable is passed in a request to Chainlink. It transmits how many random numbers need to be received from the chainlink. The project is designed in such a way that we always request 1 random number. Therefore, there is no point in storing the number 1 in a variable and there is no point in changing this value either. And remove public keyword, because we know that value of constant is 1.
```diff
- uint32 public numWords = 1;
+ uint32 constant numWords = 1;

-    function updateAdditionalData(uint64 _s_subscriptionId, uint32 _numWords, uint16 _requestConfirmations) public FunctionAdminRequired(this.updateAdditionalData.selector){
+    function updateAdditionalData(uint64 _s_subscriptionId, uint16 _requestConfirmations) public FunctionAdminRequired(this.updateAdditionalData.selector){
        s_subscriptionId = _s_subscriptionId;
-       numWords = _numWords;
        requestConfirmations = _requestConfirmations;
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L86-L90
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L29

### [G-3] Public function calls internal only, could be marked as internal
Internal functions require less gas when deploying a contract. This applies in two places: RandomizerVRF and RandomizerRNG
```diff
- function requestRandomWords(uint256 tokenid) public {
+ function requestRandomWords(uint256 tokenid) internal {
        require(msg.sender == gencore);
       ...
    }

function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
        require(msg.sender == gencore);
        tokenIdToCollection[_mintIndex] = _collectionID;
        requestRandomWords(_mintIndex);
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L52-L75
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L40-L46

### [G-4] Store array of words in memory more expensive, than read word from storage for user.
The user spends a lot of gas writing a large array into memory. It will be cheaper if you write this array to storage during deployment. In this case, reading from storage will be cheaper than writing a huge array into memory. Yes, deployment will be more expensive, but then such optimization will be cheaper for the user
```diff
contract randomPool {

+ string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", 
+        "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine", 
+        "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian", 
+        "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum", 
+        "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango", 
+        "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple", 
+        "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma", 
+        "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];

function getWord(uint256 id) private pure returns (string memory) {
        
-        // array storing the words list
-        string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange", 
-        "Blueberry", "Boysenberry", "Breadfruit", "Brush Cherry", "Canary Melon", "Cantaloupe", "Carambola", "Casaba Melon", "Cherimoya", "Cherry", "Clementine", 
-        "Cloudberry", "Coconut", "Cranberry", "Crenshaw Melon", "Cucumber", "Currant", "Curry Berry", "Custard Apple", "Damson Plum", "Date", "Dragonfruit", "Durian", 
-        "Eggplant", "Elderberry", "Feijoa", "Finger Lime", "Fig", "Gooseberry", "Grapes", "Grapefruit", "Guava", "Honeydew Melon", "Huckleberry", "Italian Prune Plum", 
-        "Jackfruit", "Java Plum", "Jujube", "Kaffir Lime", "Kiwi", "Kumquat", "Lemon", "Lime", "Loganberry", "Longan", "Loquat", "Lychee", "Mammee", "Mandarin", "Mango", 
-        "Mangosteen", "Mulberry", "Nance", "Nectarine", "Noni", "Olive", "Orange", "Papaya", "Passion fruit", "Pawpaw", "Peach", "Pear", "Persimmon", "Pineapple", 
-        "Plantain", "Plum", "Pomegranate", "Pomelo", "Prickly Pear", "Pulasan", "Quine", "Rambutan", "Raspberries", "Rhubarb", "Rose Apple", "Sapodilla", "Satsuma", 
-        "Soursop", "Star Apple", "Star Fruit", "Strawberry", "Sugar Apple", "Tamarillo", "Tamarind", "Tangelo", "Tangerine", "Ugli", "Velvet Apple", "Watermelon"];
        
        if (id==0) {
          
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
        }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L13-L26

### [G-5] Change order of variables in structure for use less storage slots
We can place the variables in the structure in a different order so that the structure takes up 2 slots instead of 3 per storage. Because address and bool can fit into 1 slot
```diff
    struct auctionInfoStru {
-        address bidder; // 1st slot
-        uint256 bid;    // 2nd slot
-        bool status;    // 3rd slot
+        address bidder; // 1st slot
+        bool status;    // 1st slot
+        uint256 bid;    // 2nd slot
    }
```
The same reordering could be done in structures MinterContract.collectionPrimaryAddresses, MinterContract.collectionSecondaryAddresses
```diff
	struct collectionPrimaryAddresses {
        address primaryAdd1;
        address primaryAdd2;
        address primaryAdd3;
+       bool status;
        uint256 add1Percentage;
        uint256 add2Percentage;
        uint256 add3Percentage;
-       bool status;
    }

  struct collectionSecondaryAddresses {
        address secondaryAdd1;
        address secondaryAdd2;
        address secondaryAdd3;
+       bool status;
        uint256 add1Percentage;
        uint256 add2Percentage;
        uint256 add3Percentage;
-       bool status;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L43-L47
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L73-L81
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L98-L106

### [G-6] No need to temp structure in memory
We can pass values directly to push function
```diff
   function participateToAuction(uint256 _tokenid) public payable {
        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);
-        auctionInfoStru memory newBid = auctionInfoStru(msg.sender, msg.value, true);
-        auctionInfoData[_tokenid].push(newBid);
+        auctionInfoData[_tokenid].push(auctionInfoStru(msg.sender, msg.value, true));
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L57-L61


### [G-7] Iteration through the array must be done from the end
Active bets in the array are ordered in ascending order. Therefore, to calculate the highest bet, it is better to go through the cycle from the end, this will be faster and use less gas
```diff
 function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
-        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
-           uint256 highBid = 0;
-            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
-                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
-                    highBid = auctionInfoData[_tokenid][i].bid;
-                    index = i;
-                }
-            }
-            if (auctionInfoData[_tokenid][index].status == true) {
-                return highBid;
-            } else {
-                return 0;
-            }
-        } else {
-            return 0;
-        }

+       for (uint256 i=auctionInfoData[_tokenid].length-1; i !=0 ; --i) {
+               if (auctionInfoData[_tokenid][i].status == true) {
+                   return auctionInfoData[_tokenid][i].bid;
+               }
+       }
	}
```
The same fix could be applied to function returnHighestBidder()
```diff
 function returnHighestBidder(uint256 _tokenid) public view returns (address) {
-        uint256 highBid = 0;
         uint256 index;
-        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
-            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
-                index = i;
-            }
-        }
+	     for (uint256 i=auctionInfoData[_tokenid].length-1; i !=0 ; --i) {
+			if (auctionInfoData[_tokenid][i].status == true) {
+                   index = i;
+                   break;
+               }
+		}	
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87-L100

### [G-8] Don't go through the array twice to find the highest bid and its author
To determine the highest bidder and highest bid, 2 functions are called, each of which goes through a cycle. You can go through the loop once and additionally return the index of the element. Let's add an additional return argument to the returnHighestBidder function - index
```diff
    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
-       uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
-       address highestBidder = returnHighestBidder(_tokenid);
+       (address highestBidder, uint index) = returnHighestBidder(_tokenid);
+       uint256 highestBid = auctionInfoData[_tokenid][index].bid;
   ... 
   }
   
-   function returnHighestBidder(uint256 _tokenid) public view returns (address) {
+   function returnHighestBidder(uint256 _tokenid) public view returns (address, uint) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
-               return auctionInfoData[_tokenid][index].bidder;
+               return (auctionInfoData[_tokenid][index].bidder, index);
            } else {
                revert("No Active Bidder");
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L104-L109
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L87-L100

### [G-9] Cache structure data in memory variable
```diff
   function cancelBid(uint256 _tokenid, uint256 index) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
+		auctionInfoStru memory auctionInfo = auctionInfoData[_tokenid][index];    
-       require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
+       require(auctionInfo.bidder == msg.sender && auctionInfo.status == true);
        auctionInfoData[_tokenid][index].status = false;
-       (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
+       (bool success, ) = payable(auctionInfo.bidder).call{value: auctionInfo.bid}("");
        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
    }
```
The same trick could be applied for function cancelAllBids()
```diff
    function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
+				auctionInfoStru memory auctionInfo = auctionInfoData[_tokenid][i];                
-               (bool success, ) = payable(auctionInfo.bidder).call{value: auctionInfo.bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L124-L130
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L134-L143

### [G-10] Dont need call viewCirSupply() in each iteration
Before starting to loop through the collection, we ask for the minimum possible index for the collection and the current supply. And we will use this data when going through the loop. After each pass, we increase this data by 1. This allows us not to send requests to the contract core
```diff
  function airDropTokens(address[] memory _recipients, string[] memory _tokenData, uint256[] memory _saltfun_o, uint256 _collectionID, uint256[] memory _numberOfTokens) public FunctionAdminRequired(this.airDropTokens.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
+       uint inMin = gencore.viewTokensIndexMin(_collectionID);
+       uint curSupply = gencore.viewCirSupply(_collectionID);
        for (uint256 y=0; y< _recipients.length; y++) {
-           collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
+           collectionTokenMintIndex = inMin + curSupply + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
-               uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
+               gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
+               mintIndex++;
+               curSupply++;
            }
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181-L192

### [G-11] There is no need to cache a calldata type variable in memory
Memory caching is used when a value is read from storage many times. But in this case, our variable is in calldata area, so we don’t need to waste gas writing and reading values from memory
```diff
    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
-       uint256 col = _collectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;
-       if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
+        if (block.timestamp >= collectionPhases[_collectionID].allowlistStartTime && block.timestamp <= collectionPhases[_collectionID].allowlistEndTime) {
            phase = 1;
            bytes32 node;
            if (_delegator != 0x0000000000000000000000000000000000000000) {
                bool isAllowedToMint;
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
                if (isAllowedToMint == false) {
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);    
                }
                require(isAllowedToMint == true, "No delegation");
                node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");
                mintingAddress = _delegator;
            } else {
                node = keccak256(abi.encodePacked(msg.sender, _maxAllowance, tokData));
                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, msg.sender) + _numberOfTokens, "AL limit");
                mintingAddress = msg.sender;
            }
-           require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');
+           require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[_collectionID].merkleRoot, node), 'invalid proof');
-       } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
+       } else if (block.timestamp >= collectionPhases[_collectionID].publicStartTime && block.timestamp <= collectionPhases[_collectionID].publicEndTime) {
            phase = 2;
-           require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
+           require(_numberOfTokens <= gencore.viewMaxAllowance(_collectionID), "Change no of tokens");
-           require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
+            require(gencore.retrieveTokensMintedPublicPerAddress(_collectionID, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(_collectionID), "Max");
            mintingAddress = msg.sender;
            tokData = '"public"';
        } else {
            revert("No minting");
        }
        uint256 collectionTokenMintIndex;
-       collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
+       collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens - 1;
-       require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
+       require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
-       require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
+       require(msg.value >= (getPrice(_collectionID) * _numberOfTokens), "Wrong ETH");
        for(uint256 i = 0; i < _numberOfTokens; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
-           gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
+           gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, _collectionID, phase);
        }
-       collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
+       collectionTotalAmount[col] = collectionTotalAmount[_collectionID] + msg.value;
        // control mechanism for sale option 3
-       if (collectionPhases[col].salesOption == 3) {
+       if (collectionPhases[_collectionID].salesOption == 3) {
            uint timeOfLastMint;
-           if (lastMintDate[col] == 0) {
+           if (lastMintDate[_collectionIDcol] == 0) {
                // for public sale set the allowlist the same time as publicsale
-               timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;
+               timeOfLastMint = collectionPhases[_collectionID].allowlistStartTime - collectionPhases[_collectionID].timePeriod;
            } else {
-               timeOfLastMint =  lastMintDate[col];
+               timeOfLastMint =  lastMintDate[_collectionID];
            }
            // uint calculates if period has passed in order to allow minting
-           uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
+           uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;
            // users are able to mint after a day passes
            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
-           lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
+           lastMintDate[col] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L196C1-L254C6

### [G-12] Unnecessary token limit check
The check will be performed at the end of the function. No need to duplicate check
```diff
function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        ...
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            phase = 2;
            //
            // Look at the end of the function
-           require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
            mintingAddress = msg.sender;
            tokData = '"public"';
        } else {
            revert("No minting");
        }
        uint256 collectionTokenMintIndex;
+       //
+       // Here, next 2 lines
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

		...
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L196C1-L254C6

### [G-13] No need to request viewCirSupply every time, function can calculate it byself
We won’t send a request at each iteration, but we can calculate the index ourselves, increase it by 1
```diff
function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
	...
+   uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
	for(uint256 i = 0; i < _numberOfTokens; i++) {
-           uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
+           mintIndex++;
        }
   ...
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L234-L237

### [G-14] Multiplying by 1 doesn't change anything
Any number multiplied by 1 remains the same
```diff
	 function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
   		...
-       require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
+       require(msg.value >= getPrice(col), "Wrong ETH");
        ...
     }
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L361

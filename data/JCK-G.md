


## Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| Avoid Unnecessary Public Variables   |  14  | 308000 |
|[G-02]| Possible Optimizations in MinterContract.sol   |  3  | |
|[G-03]| Possible Optimization 1   |  2  | |
|[G-04]| Possible Optimization    |  1  | |
|[G-05]| Possible Optimizations NextGenAdmins.sol   |  1  | |
|[G-06]| Possible Optimizations   |  32  | |
|[G-07]| Possible Optimization AuctionDemo.sol   |  1  | |
|[G-08]| Pack structs by putting data types that can fit together next to each other   |  1  | |
|[G-09]| Don’t cache calls that are only used once   |  5  | |
|[G-10]| Optimize the function burnOrSwapExternalToMint()   |  1  | |
|[G-11]| public functions not called by the contract should be declared external instead   |  34  | |
|[G-12]| Optimize Gas by Using Do-While Loops   |  2  | |
|[G-13]| Enable --via-ir for Potential Gas Savings Through Cross-Function Optimizations   |  8  | |
|[G-14]| Optimize External Calls with Assembly for Memory Efficiency   |  9  | |
|[G-15]| Consider Using Solady's Gas Optimized Lib for Math   |  2  | |
|[G-16]| Optimize Unsigned Integer Comparison With Zero  |  6  | |
|[G-17]| Use uint256(1)/uint256(2) instead for true and false boolean states  |  18  | 18000 |
|[G-18]| Cache external calls outside of loop to avoid re-calling function on each iteration  |  2  | |
|[G-19]| Use hardcoded address instead of address(this)  |  3  | |
|[G-20]| Use assembly for loops  |  5  | |
|[G-21]| Counting down in for statements is more gas efficient  |  8  | |

## [G-01] Avoid Unnecessary Public Variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

If you do not require other contracts to read these variables, consider making them private or internal.

```solidity
file: main/smart-contracts/AuctionDemo.sol

26   IMinterContract public minter;

27   INextGenAdmins public adminsContract;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L26

```solidity
file: main/smart-contracts/MinterContract.sol

118   INextGenCore public gencore;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L118

```solidity
file: main/smart-contracts/NextGenCore.sol

26   uint256 public newCollectionIndex;

105  address public minterContract;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L26

```solidity
file: main/smart-contracts/RandomizerNXT.sol

20    IXRandoms public randoms;

22    INextGenCore public gencoreContract;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L20

```solidity
file:  main/smart-contracts/RandomizerRNG.sol

22   INextGenCore public gencoreContract;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L22

```solidity
file: main/smart-contracts/RandomizerVRF.sol

22  VRFCoordinatorV2Interface public COORDINATOR;

26  bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;

27  uint32 public callbackGasLimit = 40000;

28  uint16 public requestConfirmations = 3;

29  uint32 public numWords = 1;

36  INextGenCore public gencoreContract;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L22

## [G-02] Possible Optimizations in MinterContract.sol

In the mint, burnToMint and burnOrSwapExternalToMint function, the global variable msg.value is verified against the function parameter following an extensive operation. Optimizing gas consumption can be achieved by performing the verification of the msg.value variable before the commencement of this substantial operation.

```solidity
file: main/smart-contracts/MinterContract.sol

233   require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");

266   require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");

361  require(msg.value >= (getPrice(col) * 1), "Wrong ETH");

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L233

Estimated gas saved:
This optimization could save around 5000 to 10000 gas per function operation , depending on the gas cost of the mint operation. 


## [G-03] Possible Optimization 1

The returnHighestBid and cancelAllBids function uses a loop to update auctionInfoData, auctionInfoData for each tokenId. However, the tokenId is not checked for uniqueness, which means that the same tokenId could be updated multiple times in a single transaction, wasting gas. Adding a check to ensure that each tokenId is unique could save gas.

```solidity
file:  main/smart-contracts/AuctionDemo.sol

69      for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }

136      for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69-L74

Estimated gas saved:
This optimization could save around 5000 to 10000 gas per transaction, depending on the number of tokenIds that are not unique. This is because the SSTORE opcode, which is used to update storage, is one of the most expensive operations in terms of gas cost. By reducing the number of unnecessary SSTORE operations, we can save a significant amount of gas.

## [G-04] Possible Optimization 

The updateImagesAndAttributes function uses a loop to update tokenImageAndAttributes for each tokenId. However, the tokenId is not checked for uniqueness, which means that the same tokenId could be updated multiple times in a single transaction, wasting gas. Adding a check to ensure that each tokenId is unique could save gas.

```solidity
file:  main/smart-contracts/NextGenCore.sol

281      function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
        for (uint256 x; x < _tokenId.length; x++) {
            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
            _requireMinted(_tokenId[x]);
            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
        }
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L281-L288

## [G-05] Possible Optimizations NextGenAdmins.sol

In the "registerBatchFunctionAdmin" function, there is currently no validation in place to verify the authenticity and validity of the admin address. Prior to initiating the loop iteration, conducting a validation check to confirm the legitimacy of the admin address can result in a gas-saving measure.

```solidity
file: main/smart-contracts/NextGenAdmins.sol

50    function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
        for (uint256 i=0; i<_selector.length; i++) {
            functionAdmin[_address][_selector[i]] = _status;
        }
    }


```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L50-L54

## [G-06] Possible Optimizations

###  General Optimization

Reducing the number of state variable updates can save gas. In Ethereum, every storage operation costs a significant amount of gas. Therefore, optimizing the contract to minimize storage operations can lead to substantial gas savings.

### Possible Optimization 1

In the returnHighestBid, returnHighestBidder, claimAuction,cancelBid  and cancelAllBids functions, the auctionInfoData[_tokenid] mapping is updated 8,7,6,or 5 time. This can be optimized to a single update, which would save gas.


```solidity
file:  main/smart-contracts/AuctionDemo.sol

65      function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }

87    function returnHighestBidder(uint256 _tokenid) public view returns (address) {
        uint256 highBid = 0;
        uint256 index;
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }
        if (auctionInfoData[_tokenid][index].status == true) {
                return auctionInfoData[_tokenid][index].bidder;
            } else {
                revert("No Active Bidder");
        }
    }

104  function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
        auctionClaim[_tokenid] = true;
        uint256 highestBid = returnHighestBid(_tokenid);
        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
        address highestBidder = returnHighestBidder(_tokenid);
        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
        }
    }

124 function cancelBid(uint256 _tokenid, uint256 index) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);
        auctionInfoData[_tokenid][index].status = false;
        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);
    }

134   function cancelAllBids(uint256 _tokenid) public {
        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
                auctionInfoData[_tokenid][i].status = false;
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
            } else {}
        }
    }



```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L65-L83


Estimated gas saved:
This optimization reduces the number of storage operations from 8,7,6 or 5 to 1 in each functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions.


### Possible Optimization 2

In the payArtist functions, the collectionTotalAmount[_collectionID]  mapping is updated 3 time. This can be optimized to a single update, which would save gas.

```solidity 
file:  main/smart-contracts/MinterContract.sol

415      function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        collectionTotalAmount[_collectionID] = 0;
     
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L415-L420

Estimated gas saved:
This optimization reduces the number of storage operations from 3 to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions.


### Possible Optimization 3

In the mint and mintAndAuction functions, the lastMintDate mapping is updated 3 time. This can be optimized to a single update, which would save gas.


```solidity
file:  smart-contracts/MinterContract.sol

240           if (collectionPhases[col].salesOption == 3) {
            uint timeOfLastMint;
            if (lastMintDate[col] == 0) {
                // for public sale set the allowlist the same time as publicsale
                timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;
            } else {
                timeOfLastMint =  lastMintDate[col];
            }
            // uint calculates if period has passed in order to allow minting
            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
            // users are able to mint after a day passes
            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
        }
    }

276       function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
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
        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
        mintToAuctionData[mintIndex] = _auctionEndTime;
        mintToAuctionStatus[mintIndex] = true;
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L240-L253

Estimated gas saved:
This optimization reduces the number of storage operations from 3 to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions

### Possible Optimization 4

In the following functions, the collectionPhases mapping is updated more, more time . This can be optimized to a single update, which would save gas.


```solidity
file:  main/smart-contracts/MinterContract.sol

157    function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
        collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
        collectionPhases[_collectionID].rate = _rate;
        collectionPhases[_collectionID].timePeriod = _timePeriod;
        collectionPhases[_collectionID].salesOption = _salesOption;
        collectionPhases[_collectionID].delAddress = _delAddress;
        setMintingCosts[_collectionID] = true;
    }

170    function setCollectionPhases(uint256 _collectionID, uint _allowlistStartTime, uint _allowlistEndTime, uint _publicStartTime, uint _publicEndTime, bytes32 _merkleRoot) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
        collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
        collectionPhases[_collectionID].merkleRoot = _merkleRoot;
        collectionPhases[_collectionID].publicStartTime = _publicStartTime;
        collectionPhases[_collectionID].publicEndTime = _publicEndTime;
    }


195       function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
        uint256 col = _collectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;
        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
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
            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            phase = 2;
            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
            mintingAddress = msg.sender;
            tokData = '"public"';
        } else {
            revert("No minting");
        }
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
        for(uint256 i = 0; i < _numberOfTokens; i++) {
            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
        }
        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
        // control mechanism for sale option 3
        if (collectionPhases[col].salesOption == 3) {
            uint timeOfLastMint;
            if (lastMintDate[col] == 0) {
                // for public sale set the allowlist the same time as publicsale
                timeOfLastMint = collectionPhases[col].allowlistStartTime - collectionPhases[col].timePeriod;
            } else {
                timeOfLastMint =  lastMintDate[col];
            }
            // uint calculates if period has passed in order to allow minting
            uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
            // users are able to mint after a day passes
            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");
            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
        }
    }


276   function mintAndAuction(address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint _auctionEndTime) public FunctionAdminRequired(this.mintAndAuction.selector) {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
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
        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));
        mintToAuctionData[mintIndex] = _auctionEndTime;
        mintToAuctionStatus[mintIndex] = true;
    }

326      function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
        address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
        if (msg.sender != ownerOfToken) {
            bool isAllowedToMint;
            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);
            if (isAllowedToMint == false) {
            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);    
            }
            require(isAllowedToMint == true, "No delegation");
        }
        require(_tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1], "Token id does not match");
        IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
        uint256 col = _mintCollectionID;
        address mintingAddress;
        uint256 phase;
        string memory tokData = _tokenData;
        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
            phase = 1;
            bytes32 node;
            node = keccak256(abi.encodePacked(_tokenId, tokData));
            mintingAddress = ownerOfToken;
            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');            
        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            phase = 2;
            mintingAddress = ownerOfToken;
            tokData = '"public"';
        } else {
    
494  function retrieveCollectionPhases(uint256 _collectionID) public view returns(uint, uint, bytes32, uint, uint){
        return (collectionPhases[_collectionID].allowlistStartTime, collectionPhases[_collectionID].allowlistEndTime, collectionPhases[_collectionID].merkleRoot, collectionPhases[_collectionID].publicStartTime, collectionPhases[_collectionID].publicEndTime);
    }

500  function retrieveCollectionMintingDetails(uint256 _collectionID) public view returns(uint256, uint256, uint256, uint256, uint8, address){
        return (collectionPhases[_collectionID].collectionMintCost, collectionPhases[_collectionID].collectionEndMintCost, collectionPhases[_collectionID].rate, collectionPhases[_collectionID].timePeriod, collectionPhases[_collectionID].salesOption, collectionPhases[_collectionID].delAddress);
    }

530      function getPrice(uint256 _collectionId) public view returns (uint256) {
        uint tDiff;
        if (collectionPhases[_collectionId].salesOption == 3) {
            // increase minting price by mintcost / collectionPhases[_collectionId].rate every mint (1mint/period)
            // to get the price rate needs to be set
            if (collectionPhases[_collectionId].rate > 0) {
                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
            } else {
                return collectionPhases[_collectionId].collectionMintCost;
            }
        } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
            // decreases exponentially every time period
            // collectionPhases[_collectionId].timePeriod sets the time period for decreasing the mintcost
            // if just public mint set the publicStartTime = allowlistStartTime
            // if rate = 0 exponetialy decrease
            // if rate is set the linear decrase each period per rate
            tDiff = (block.timestamp - collectionPhases[_collectionId].allowlistStartTime) / collectionPhases[_collectionId].timePeriod;
            uint256 price;
            uint256 decreaserate;
            if (collectionPhases[_collectionId].rate == 0) {
                price = collectionPhases[_collectionId].collectionMintCost / (tDiff + 1);
                decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
            } else {
                if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
                    price = collectionPhases[_collectionId].collectionMintCost - (tDiff * collectionPhases[_collectionId].rate);
                } else {
                    price = collectionPhases[_collectionId].collectionEndMintCost;
                }
            }
            if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
                return price - decreaserate; 
            } else {
                return collectionPhases[_collectionId].collectionEndMintCost;
            }
        } else {
            // fixed price
            return collectionPhases[_collectionId].collectionMintCost;
        }
    }
     

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157-L166 

Estimated gas saved:
This optimization reduces the number of storage operations from more time  to 1 in functions. Given that a storage operation costs 20,0000 gas, this optimization could save approximately 20,0000 gas per call to these functions

### Possible Optimization 5

In the proposePrimaryAddressesAndPercentages, payArtist and retrievePrimaryAddressesAndPercentages functions, the collectionArtistPrimaryAddresses mapping is updated more time. This can be optimized to a single update, which would save gas.

```solidity
file:  main/smart-contracts/MinterContract.sol

380    function proposePrimaryAddressesAndPercentages(uint256 _collectionID, address _primaryAdd1, address _primaryAdd2, address _primaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposePrimaryAddressesAndPercentages.selector) {
        require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd1 = _primaryAdd1;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd2 = _primaryAdd2;
        collectionArtistPrimaryAddresses[_collectionID].primaryAdd3 = _primaryAdd3;
        collectionArtistPrimaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistPrimaryAddresses[_collectionID].add3Percentage = _add3Percentage;
        collectionArtistPrimaryAddresses[_collectionID].status = false;
    }
415     function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");
        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");
        uint256 royalties = collectionTotalAmount[_collectionID];
        collectionTotalAmount[_collectionID] = 0;
        address tm1 = _team1;
        address tm2 = _team2;
        uint256 colId = _collectionID;
        uint256 artistRoyalties1;
        uint256 artistRoyalties2;
        uint256 artistRoyalties3;
        uint256 teamRoyalties1;
        uint256 teamRoyalties2;
        artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
        artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
        artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
        teamRoyalties1 = royalties * _teamperc1 / 100;
        teamRoyalties2 = royalties * _teamperc2 / 100;
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);
        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
        emit PayTeam(tm1, success4, teamRoyalties1);
        emit PayTeam(tm2, success5, teamRoyalties2);
    }

476    function retrievePrimaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){
        return (collectionArtistPrimaryAddresses[_collectionID].primaryAdd1, collectionArtistPrimaryAddresses[_collectionID].primaryAdd2, collectionArtistPrimaryAddresses[_collectionID].primaryAdd3, collectionArtistPrimaryAddresses[_collectionID].add1Percentage, collectionArtistPrimaryAddresses[_collectionID].add2Percentage, collectionArtistPrimaryAddresses[_collectionID].add3Percentage, collectionArtistPrimaryAddresses[_collectionID].status);
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L380-L390

Estimated gas saved:
This optimization reduces the number of storage operations from more time  to 1 in functions. Given that a storage operation costs 20,0000 gas, this optimization could save approximately 20,0000 gas per call to these functions

### Possible Optimization 6

In the proposeSecondaryAddressesAndPercentages and retrieveSecondaryAddressesAndPercentages functions, the collectionArtistSecondaryAddresses mapping is updated more time. This can be optimized to a single update, which would save gas.

```solidity
file:

394     function proposeSecondaryAddressesAndPercentages(uint256 _collectionID, address _secondaryAdd1, address _secondaryAdd2, address _secondaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposeSecondaryAddressesAndPercentages.selector) {
        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1 = _secondaryAdd1;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2 = _secondaryAdd2;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3 = _secondaryAdd3;
        collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
        collectionArtistSecondaryAddresses[_collectionID].status = false;
    }

488    function retrieveSecondaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){
        return (collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1, collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2, collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3, collectionArtistSecondaryAddresses[_collectionID].add1Percentage, collectionArtistSecondaryAddresses[_collectionID].add2Percentage, collectionArtistSecondaryAddresses[_collectionID].add3Percentage, collectionArtistSecondaryAddresses[_collectionID].status);
    }


```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L769-L807

Estimated gas saved:
This optimization reduces the number of storage operations from more time  to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions


### Possible Optimization 7

In the createCollection and updateCollectionInfo functions, the collectionInfo mapping is updated more time. This can be optimized to a single update, which would save gas.

```solidity
file:

130      function createCollection(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public FunctionAdminRequired(this.createCollection.selector) {
        collectionInfo[newCollectionIndex].collectionName = _collectionName;
        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
        isCollectionCreated[newCollectionIndex] = true;
        newCollectionIndex = newCollectionIndex + 1;
    }

238  function updateCollectionInfo(uint256 _collectionID, string memory _newCollectionName, string memory _newCollectionArtist, string memory _newCollectionDescription, string memory _newCollectionWebsite, string memory _newCollectionLicense, string memory _newCollectionBaseURI, string memory _newCollectionLibrary, uint256 _index, string[] memory _newCollectionScript) public CollectionAdminRequired(_collectionID, this.updateCollectionInfo.selector) {
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
         if (_index == 1000) {
            collectionInfo[_collectionID].collectionName = _newCollectionName;
            collectionInfo[_collectionID].collectionArtist = _newCollectionArtist;
            collectionInfo[_collectionID].collectionDescription = _newCollectionDescription;
            collectionInfo[_collectionID].collectionWebsite = _newCollectionWebsite;
            collectionInfo[_collectionID].collectionLicense = _newCollectionLicense;
            collectionInfo[_collectionID].collectionLibrary = _newCollectionLibrary;
            collectionInfo[_collectionID].collectionScript = _newCollectionScript;
        } else if (_index == 999) {
            collectionInfo[_collectionID].collectionBaseURI = _newCollectionBaseURI;
        } else {
            collectionInfo[_collectionID].collectionScript[_index] = _newCollectionScript[0];
        }
    }

426  function retrieveCollectionInfo(uint256 _collectionID) public view returns(string memory, string memory, string memory, string memory, string memory, string memory){
        return (collectionInfo[_collectionID].collectionName, collectionInfo[_collectionID].collectionArtist, collectionInfo[_collectionID].collectionDescription, collectionInfo[_collectionID].collectionWebsite, collectionInfo[_collectionID].collectionLicense, collectionInfo[_collectionID].collectionBaseURI);
    } 

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L130-L141

Estimated gas saved:
This optimization reduces the number of storage operations from more time  to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions

### Possible Optimization 8

In the setCollectionData, airDropTokens, mint, burnToMint and setFinalSupply functions, the collectionAdditionalData mapping is updated more time. This can be optimized to a single update, which would save gas.


```solidity
file:  main/smart-contracts/NextGenCore.sol

147      function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
            collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
            collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
            collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
            wereDataAdded[_collectionID] = true;
        } else if (artistSigned[_collectionID] == false) {
            collectionAdditionalData[_collectionID].collectionArtistAddress = _collectionArtistAddress;
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
        } else {
            collectionAdditionalData[_collectionID].maxCollectionPurchases = _maxCollectionPurchases;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
        }
    }

178   function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }

189   function mint(uint256 mintIndex, address _mintingAddress , address _mintTo, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID, uint256 phase) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
            if (phase == 1) {
                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
            } else {
                tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
            }
        }
    }

213 function burnToMint(uint256 mintIndex, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o, address burner) external {
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

307  function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147-L166

Estimated gas saved:
This optimization reduces the number of storage operations from more time  to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions

### Possible Optimization 9

In the tokenURI and retrieveGenerativeScript functions, the tokenIdsToCollectionIds mapping is updated more time. This can be optimized to a single update, which would save gas.


```solidity
file:  main/smart-contracts/NextGenCore.sol

343     function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
        _requireMinted(tokenId);
        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
        } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
        }
        else {
            string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));
            string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));
            return _uri;
        }
    }

450  function retrieveGenerativeScript(uint256 tokenId) public view returns(string memory){
        _requireMinted(tokenId);
        string memory scripttext;
        for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
            scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 
        }
        return string(abi.encodePacked("let hash='",Strings.toHexString(uint256(tokenToHash[tokenId]), 32),"';let tokenId=",tokenId.toString(),";let tokenData=[",tokenData[tokenId],"];", scripttext));
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L343-L357

Estimated gas saved:
This optimization reduces the number of storage operations from more time  to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions

### Possible Optimization 10

In the fulfillRandomWords functions, the tokenIdToCollection mapping is updated 3 time. This can be optimized to a single update, which would save gas.

```solidity
file: smart-contracts/RandomizerRNG.sol

48   function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers,requestToToken[id])));
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L48-L50

Estimated gas saved:
This optimization reduces the number of storage operations from 3 to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions

### Possible Optimization 11

In the fulfillRandomWords functions, the tokenIdToCollection mapping is updated 3 time. This can be optimized to a single update, which would save gas.


```solidity
file: main/smart-contracts/RandomizerVRF.sol

65  function fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords) internal override {
        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[_requestId]], requestToToken[_requestId], bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])));
        emit RequestFulfilled(_requestId, _randomWords);
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L65-L69

Estimated gas saved:
This optimization reduces the number of storage operations from 3 to 1 in functions. Given that a storage operation costs 20,000 gas, this optimization could save approximately 20,000 gas per call to these functions

## [G-07] Possible Optimization AuctionDemo.sol


cache the value of  auctionInfoData[_tokenid].length to save gas 

```solidity
file:  main/smart-contracts/AuctionDemo.sol

65     function returnHighestBid(uint256 _tokenid) public view returns (uint256) {
        uint256 index;
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
        } else {
            return 0;
        }
    }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L65-L83

Estimated gas saved:
This optimization could save around 500 to 1000 gas per transaction, depending on the gas cost of the returnHighestBid function call.

## [G-08] Pack structs by putting data types that can fit together next to each other

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot. This saves gas when writing to storage ~20000 gas

```solidity
file: main/smart-contracts/NextGenCore.sol

44  struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
        uint256 reservedMinTokensIndex;
        uint256 reservedMaxTokensIndex;
        uint setFinalSupplyTimeAfterMint;
        address randomizerContract;
        IRandomizer randomizer;
    }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L44-L54

## [G-09] Don’t cache calls that are only used once

```solidity
file: main/smart-contracts/MinterContract.sol

463  address admin = adminsContract.owner();

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L463

Estimated gas saved:
This optimization could save around 10 to 20 gas, 


```solidity
file: main/smart-contracts/RandomizerRNG.sol

81   address admin = adminsContract.owner();

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L81

Estimated gas saved:
This optimization could save around 10 to 20 gas, 


```solidity
file: main/smart-contracts/AuctionDemo.sol

108  address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L108

Estimated gas saved:
This optimization could save around 10 to 20 gas, 


```solidity
file: main/smart-contracts/XRandoms.sol

36    uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;

41    uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L36

Estimated gas saved:
This optimization could save around 2000 to 2484 gas, in bothe randomNumber and randomWord function.


## [G-10] Optimize the function burnOrSwapExternalToMint()

```solidity
file:  main/smart-contracts/MinterContract.sol

332    bool isAllowedToMint;
      isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);
      if (isAllowedToMint == false) {
      isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);    
      }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L332-L338

The function above, only mints if the isAllowedToMint is met. Before checking the status of whether the isAllowedToMint is met or not, we making a state readisAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);, which is quite expensive. We then proceed to check if the isAllowedToMint is met and execute the function if the isAllowedToMint is indeed met. If not met, we would not execute anything, but we would still have made the state read, as it’s being read before the check.

We can refactor the function to have the check before to avoid wasting gas in case isAllowedToMint is not met:

## [G-11] public functions not called by the contract should be declared external instead

when a function is declared as public, it is generated with an internal and an external interface. This means the function can be called both internally (within the contract) and externally (by other contracts or accounts). However, if a public function is never called internally and is only expected to be invoked externally, it is more gas-efficient to explicitly declare it as external.


```solidity
file: main/smart-contracts/AuctionDemo.sol

57   function participateToAuction(uint256 _tokenid) public payable {
  
104  function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){

124  function cancelBid(uint256 _tokenid, uint256 index) public {

134  function cancelAllBids(uint256 _tokenid) public {

147  function returnBids(uint256 _tokenid) public view returns(auctionInfoStru[] memory) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57

```solidity
file: smart-contracts/MinterContract.sol

470   function retrievePrimarySplits(uint256 _collectionID) public view returns(uint256, uint256){

476   function retrievePrimaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){

482   function retrieveSecondarySplits(uint256 _collectionID) public view returns(uint256, uint256){

488   function retrieveSecondaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){

494   function retrieveCollectionPhases(uint256 _collectionID) public view returns(uint, uint, bytes32, uint, uint){

500   function retrieveCollectionMintingDetails(uint256 _collectionID) public view returns(uint256, uint256, uint256, uint256, uint8, address){

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L470

```solidity
file: main/smart-contracts/NextGenAdmins.sol

38   function registerAdmin(address _admin, bool _status) public onlyOwner {

44   function registerFunctionAdmin(address _address, bytes4 _selector, bool _status) public AdminRequired {

50   function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {

58   function registerCollectionAdmin(uint256 _collectionID, address _address, bool _status) public AdminRequired {

65   function retrieveGlobalAdmin(address _address) public view returns(bool) {

71   function retrieveFunctionAdmin(address _address, bytes4 _selector) public view returns(bool) {

77   function retrieveCollectionAdmin(address _address, uint256 _collectionID) public view returns(bool) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L38


```solidity
file: main/smart-contracts/NextGenCore.sol

257   function artistSignature(uint256 _collectionID, string memory _signature) public {

299   function setTokenHash(uint256 _collectionID, uint256 _mintIndex, bytes32 _hash) external {

367   function collectionFreezeStatus(uint256 _collectionID) public view returns(bool){

415   function retrieveTokensAirdroppedPerAddress(uint256 _collectionID, address _address) public view returns(uint256) {

426   function retrieveCollectionInfo(uint256 _collectionID) public view returns(string memory, string memory, string memory, string memory, string memory, string memory){

432   function retrieveCollectionLibraryAndScript(uint256 _collectionID) public view returns(string memory, string[] memory){

438   function retrieveCollectionAdditionalData(uint256 _collectionID) public view returns(address, uint256, uint256, uint256, uint, address){

444   function retrieveTokenHash(uint256 _tokenid) public view returns(bytes32){

461   function totalSupplyOfCollection(uint256 _collectionID) public view returns (uint256) {

467   function retrievetokenImageAndAttributes(uint256 _tokenId) public view returns(string memory, string memory) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L257


```solidity
file: main/smart-contracts/RandomizerNXT.sol

55   function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L55


```solidity
file: main/smart-contracts/RandomizerRNG.sol
 
53   function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L53

```solidity
file: main/smart-contracts/RandomizerVRF.sol

71   function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L71


```solidity
file: main/smart-contracts/XRandoms.sol

35  function randomNumber() public view returns (uint256){

40  function randomWord() public view returns (string memory) {

45  function returnIndex(uint256 id) public view returns (string memory) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L35

## [G-12] Optimize Gas by Using Do-While Loops

Using do-while loops instead of for loops can be more gas-efficient. Even if you add an if condition to account for the case where the loop doesn't execute at all, a do-while loop can still be cheaper in terms of gas.

```solidity
file: main/smart-contracts/NextGenAdmins.sol

51   for (uint256 i=0; i<_selector.length; i++) {
            functionAdmin[_address][_selector[i]] = _status;
        }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L51


```solidity
file: main/smart-contracts/NextGenCore.sol

453  for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
            scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 
        }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L453


## [G-13] Enable --via-ir for Potential Gas Savings Through Cross-Function Optimizations

The --via-ir command line option activates the IR-based code generator in Solidity, which is designed to enable powerful optimization passes that can span across functions. The end result may be a contract that requires less gas to execute its functions.

We recommend you enable this feature, run tests, and benchmark the gas usage of your contract to evaluate if it leads to any tangible gas savings. Experimenting with this feature could lead to a more gas-efficient contract.

8 issue instances in 8 files:

```solidity
file:  main/smart-contracts/MinterContract.sol

20  contract NextGenMinterContract is Ownable {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L20

## [G-14] Optimize External Calls with Assembly for Memory Efficiency

Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.

Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.

Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.


```solidity
file: main/smart-contracts/AuctionDemo.sol

112   IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112


```solidity
file: main/smart-contracts/MinterContract.sol

185   collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;

188   uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);

189   gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);

267   uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);

270   gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);

279   collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);

282   gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L185


```solidity
file: main/smart-contracts/RandomizerNXT.sol

58   gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L58

## [G-15] Consider Using Solady's Gas Optimized Lib for Math

Utilizing gas-optimized math functions from libraries like Solady can lead to more efficient smart contracts. This is particularly beneficial in contracts where these operations are frequently used.

```solidity
file: main/smart-contracts/MinterContract.sol

249   uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;

292   uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[_collectionID].timePeriod;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L249

## [G-16] Optimize Unsigned Integer Comparison With Zero

For unsigned integers, checking whether the integer is not equal to zero (!= 0) is less gas-intensive than checking whether it is greater than zero (> 0).

This is because the Ethereum Virtual Machine (EVM) can perform a simple bitwise operation to check if any bit is set (which directly translates to != 0), while checking for > 0 requires additional logic.

As such, when dealing with unsigned integers in Solidity, it is recommended to use the != 0 comparison for gas optimization.

```solidity
file: main/smart-contracts/AuctionDemo.sol

67   if (auctionInfoData[_tokenid].length > 0) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L67

```solidity
file: main/smart-contracts/MinterContract.sol

417   require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");

535   if (collectionPhases[_collectionId].rate > 0) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L417

```solidity
file: main/smart-contracts/NextGenAdmins.sol

59    require(_collectionID > 0, "Collection Id must be larger than 0");

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L59

```solidity
file:  main/smart-contracts/NextGenCore.sol

347    return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

350    return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L347

## [G-17] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. see source:


```solidity
file: main/smart-contracts/AuctionDemo.sol

53   mapping (uint256 => bool) public auctionClaim;

46  bool status;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L53


```solidity
file: main/smart-contracts/MinterContract.sol

35    mapping (uint256 => mapping (uint256 => bool)) public burnToMintCollections;

38    mapping (bytes32 => mapping (uint256 => bool)) public burnExternalToMintCollections;

41    mapping (uint256 => bool) private setMintingCosts;

80    bool status;

105   bool status;

115   mapping (uint256 => bool) private mintToAuctionStatus;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L35


```solidity
file: main/smart-contracts/NextGenAdmins.sol

18   mapping(address => bool) public adminPermissions;

21   mapping (address => mapping (uint256 => bool)) private collectionAdmin;

24   mapping (address => mapping (bytes4 => bool)) private functionAdmin;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L18


```solidity
file: main/smart-contracts/NextGenCore.sol

62    mapping (uint256 => bool) private isCollectionCreated; 

65    mapping (uint256 => bool) private wereDataAdded;

86    mapping (uint256 => bool) public onchainMetadata; 

98    mapping (uint256 => bool) private collectionFreeze;

101   mapping (uint256 => bool) public artistSigned; 

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L62

## [G-18] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.


### Cache viewTokensIndexMin(_collectionID) and viewTokensIndexMax(_collectionID); outside of loop to save 3 STATICCALL per loop iteration

```solidity 
file:  main/smart-contracts/MinterContract.sol

184      for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
            }
        }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L184-L191


```solidity
file: main/smart-contracts/AuctionDemo.sol

112   IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112


## [G-19] Use hardcoded address instead of address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. Refrences

```solidity
file: main/smart-contracts/MinterContract.sol

462   uint balance = address(this).balance;

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L462


```solidity
file: main/smart-contracts/RandomizerRNG.sol

42    uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));

80    uint balance = address(this).balance;\

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L42

## [G-20] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file:  main/smart-contracts/AuctionDemo.sol

69   for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                    highBid = auctionInfoData[_tokenid][i].bid;
                    index = i;
                }
            }

90    for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {
                index = i;
            }
        }

        
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69-L74


```solidity
file: main/smart-contracts/NextGenAdmins.sol

51    for (uint256 i=0; i<_selector.length; i++) {
            functionAdmin[_address][_selector[i]] = _status;
        }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L51-L53



```solidity
file: main/smart-contracts/NextGenCore.sol

282    for (uint256 x; x < _tokenId.length; x++) {
            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
            _requireMinted(_tokenId[x]);
            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
        }

453    for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
            scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i])); 
        }

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L282-L286


## [G‑02] Counting down in for statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.


```solidity
file: main/smart-contracts/AuctionDemo.sol

69   for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {

90   for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {

110  for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {

136  for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L69

```solidity
file: main/smart-contracts/MinterContract.sol

184    for (uint256 y=0; y< _recipients.length; y++) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L184

```solidity
file: main/smart-contracts/NextGenAdmins.sol

51   for (uint256 i=0; i<_selector.length; i++) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L51

```solidity
file: main/smart-contracts/NextGenCore.sol

282   for (uint256 x; x < _tokenId.length; x++) {

453   for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {

```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L282


### Test Code

```solidity

contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;
    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }
    function testGas() public {
        c0.AddNum();
        c1.AddNum();
    }
}
contract Contract0 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=0;i<=9;i++){
            _num = _num +1;
        }
        num = _num;
    }
}
contract Contract1 {
    uint256 num = 3;
    function AddNum() public {
        uint256 _num = num;
        for(uint i=9;i>=0;i--){
            _num = _num +1;
        }
        num = _num;
    }
}

```
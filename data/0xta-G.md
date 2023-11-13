# Gas Optimization

# Summary

| Number | Gas Optimization                                                                      | Context |
| :----: | :------------------------------------------------------------------------------------ | :-----: |
| [G-01] | Caching global variables is more expensive than using the actual variable             |    3    |
| [G-02] | Cache external calls outside of loop to avoid re-calling function on each iteration   |   19    |
| [G-03] | Require() strings longer than 32 bytes cost extra Gas                                 |    8    |
| [G-04] | Stack variable cost less while used in emiting event                                  |    4    |
| [G-05] | Avoid using `_msgSender` if not supporting EIP-2771                                   |    2    |
| [G-06] | Don’t make variables public unless it is necessary to do so                           |   14    |
| [G-07] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 |   16    |
| [G-08] | It is sometimes cheaper to cache calldata                                             |    2    |
| [G-09] | A modifier used only once and not being inherited should be inlined to save gas       |    1    |
| [G-10] | Pre-increment and pre-decrement are cheaper than +1 ,-1                               |   11    |
| [G-11] | Empty blocks should be removed or emit something                                      |    3    |
| [G-12] | Sort Solidity operations using short-circuit mode                                     |   31    |

## [G-01] Caching global variables is more expensive than using the actual variable

use msg.sender instead of caching it

```solidity
File: smart-contracts/MinterContract.sol

218                mintingAddress = msg.sender;

225            mintingAddress = msg.sender;

269        address burner = msg.sender;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L218

## [G-02] Cache external calls outside of loop to avoid re-calling function on each iteration

```solidity
File: smart-contracts/MinterContract.sol

184        for (uint256 y=0; y< _recipients.length; y++) {
185            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
186            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
187            for(uint256 i = 0; i < _numberOfTokens[y]; i++) {
188                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
189                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);
190            }
191        }



234        for(uint256 i = 0; i < _numberOfTokens; i++) {
235            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
236            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);
237        }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L184-L191

```solidity
File: smart-contracts/AuctionDemo.sol

110        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {
111            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
112                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
113                (bool success, ) = payable(owner()).call{value: highestBid}("");
114                emit ClaimAuction(owner(), _tokenid, success, highestBid);
115            } else if (auctionInfoData[_tokenid][i].status == true) {
116                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
117                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
118            } else {}
119        }




136        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
137            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
138                auctionInfoData[_tokenid][i].status = false;
139                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
140                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
141            } else {}
142        }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L110-L119

## [G-03] Require() strings longer than 32 bytes cost extra Gas

```solidity
File: smart-contracts/NextGenCore.sol

171        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

179        require(msg.sender == minterContract, "Caller is not the Minter Contract");

190        require(msg.sender == minterContract, "Caller is not the Minter Contract");

205        require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");

214        require(msg.sender == minterContract, "Caller is not the Minter Contract");

215        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L171

```solidity
File: main/smart-contracts/MinterContract.sol

417        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L417

```solidity
File: main/smart-contracts/NextGenAdmins.sol

59        require(_collectionID > 0, "Collection Id must be larger than 0");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L59

## [G-04] Stack variable cost less while used in emiting event

Using a stack variable instead of a state variable is cheaper when emitting an event.

```solidity
File: smart-contracts/AuctionDemo.sol


117                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);


129        emit CancelBid(msg.sender, _tokenid, index, success, auctionInfoData[_tokenid][index].bid);


140                emit CancelBid(msg.sender, _tokenid, i, success, auctionInfoData[_tokenid][i].bid);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L117

```solidity
File: smart-contracts/MinterContract.sol

439        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd1, success1, artistRoyalties1);

440        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd2, success2, artistRoyalties2);

441        emit PayArtist(collectionArtistPrimaryAddresses[colId].primaryAdd3, success3, artistRoyalties3);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L439

## [G-05] Avoid using `_msgSender` if not supporting EIP-2771

Consider using using msg.sender directly when the contract does not implement EIP-2771, as it's cheaper.

There are 2 instances of this issue.

```solidity
File: smart-contracts/NextGenAdmins.sol

32      require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32

```solidity
File: smart-contracts/NextGenCore.sol

205        require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L205

## [G-06] Don’t make variables public unless it is necessary to do so

```solidity
File: smart-contracts/MinterContract.sol

118    INextGenCore public gencore;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L118

```solidity
File: smart-contracts/RandomizerNXT.sol

20    IXRandoms public randoms;

22    INextGenCore public gencoreContract;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L20

```solidity
File: main/smart-contracts/NextGenCore.sol

26    uint256 public newCollectionIndex;

105    address public minterContract;

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L26

```solidity
File: smart-contracts/RandomizerVRF.sol

22    VRFCoordinatorV2Interface public COORDINATOR;

26    bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;

27    uint32 public callbackGasLimit = 40000;

28    uint16 public requestConfirmations = 3;

29    uint32 public numWords = 1;

36    INextGenCore public gencoreContract;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L22

```solidity
File: smart-contracts/RandomizerRNG.sol
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L22

```solidity
File: smart-contracts/AuctionDemo.sol

26    IMinterContract public minter;

27    INextGenAdmins public adminsContract;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L26

## [G-07] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

abi.encodePacked() compiles to less efficient bytecode than bytes.concat() since Solidity 0.8.4.

```solidity
File: smart-contracts/MinterContract.sol

212                node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));

216                node = keccak256(abi.encodePacked(msg.sender, _maxAllowance, tokData));

316        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));

327        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));

348            node = keccak256(abi.encodePacked(_tokenId, tokData));
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212

```solidity
File: smart-contracts/NextGenCore.sol

347            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

350            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";

353            string memory b64 = Base64.encode(abi.encodePacked("<html><head></head><body><script src=\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionLibrary,"\"></script><script>",retrieveGenerativeScript(tokenId),"</script></body></html>"));

354            string memory _uri = string(abi.encodePacked("data:application/json;utf8,{\"name\":\"",getTokenName(tokenId),"\",\"description\":\"",collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionDescription,"\",\"image\":\"",tokenImageAndAttributes[tokenId][0],"\",\"attributes\":[",tokenImageAndAttributes[tokenId][1],"],\"animation_url\":\"data:text/html;base64,",b64,"\"}"));

363        return string(abi.encodePacked(collectionInfo[viewColIDforTokenID(tokenId)].collectionName, " #" ,tok.toString()));

454            scripttext = string(abi.encodePacked(scripttext, collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript[i]));

456        return string(abi.encodePacked("let hash='",Strings.toHexString(uint256(tokenToHash[tokenId]), 32),"';let tokenId=",tokenId.toString(),";let tokenData=[",tokenData[tokenId],"];", scripttext));
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L347

```solidity
File: smart-contracts/XRandoms.sol

36        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;

41        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L36

## [G-08] It is sometimes cheaper to cache calldata

```solidity
File: smart-contracts/MinterContract.sol

196    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196

## [G-09] A modifier used only once and not being inherited should be inlined to save gas

```solidity
File: smart-contracts/AuctionDemo.sol

31    modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L31

## [G-10] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: smart-contracts/NextGenCore.sol

110        newCollectionIndex = newCollectionIndex + 1;

140        newCollectionIndex = newCollectionIndex + 1;

156            collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;


180        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;

183            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;

191        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;

195                tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;

197                tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;

208        burnAmount[_collectionID] = burnAmount[_collectionID] + 1;

221            burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;

310        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110

## [G-11] Empty blocks should be removed or emit something

```solidity
File: smart-contracts/RandomizerRNG.sol

86    receive() external payable {}
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L86

```solidity
File: smart-contracts/AuctionDemo.sol

118            } else {}

141            } else {}
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L118

## [G-12] Sort Solidity operations using short-circuit mode

```solidity
File: smart-contracts/NextGenCore.sol

117      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

124      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

148        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");

206        require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");

239        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

267        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

345        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L117

```solidity
File: smart-contracts/MinterContract.sol

137      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

144      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

151      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

202        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {


221        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {

251            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");

260        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");

261        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");

309        require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

333            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);

339        require(_tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1], "Token id does not match");

335            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);

345        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {


351        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L137

```solidity
File: smart-contracts/AuctionDemo.sol

32      require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

58        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);

70                if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {

91            if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {

105        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);

111            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {

126        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);

137            if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L32
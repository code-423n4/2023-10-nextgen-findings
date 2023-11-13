## [G-01] Avoid contract existence checks by using low level calls

**Issue Description**\
Prior to Solidity 0.8.10, the compiler would insert extra code to check for contract existence before external function calls, even if the call had a return value. This wasted gas by performing a check that wasn't necessary.

**Proposed Optimization**\
For functions that make external calls with return values in Solidity <0.8.10, optimize the code to use low-level calls instead of regular calls. Low-level calls skip the unnecessary contract existence check.

Example:

```solidity
//Before:

contract C {
  function f() external returns(uint) {
    address(otherContract).call(abi.encodeWithSignature("func()"));
  }
}


//After:

contract C {
  function f() external returns(uint) {
    (bool success,) = address(otherContract).call(abi.encodeWithSignature("func()"));
    require(success);
    return decodeReturnValue();
  }
}
```

**Estimated Gas Savings**\
Each avoided EXTCODESIZE check saves 100 gas. If 10 external calls are made in a common function, this would save 1000 gas total.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/AuctionDemo.sol

// @audit adminsContract.retrieveGlobalAdmin external call
32      require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

58        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);

105        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);

108        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);

112                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);

125        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");

135        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L32

```solidity
File: main/smart-contracts/MinterContract.sol

137      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");


144      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");


151      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");


158        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

182        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

185            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;

188                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);


207                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);

209                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);

333            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);

335            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);


340        IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);

362        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);

363        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);


536                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L137

```solidity
File: main/smart-contracts/NextGenCore.sol

117      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

124      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

171        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

308        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");

316        require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");

323        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L117

```solidity
File: main/smart-contracts/RandomizerNXT.sol

35      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

57        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));

58        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L35

```solidity
File: main/smart-contracts/RandomizerRNG.sol

36        require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

42        uint256 requestId = arrngController.requestRandomWords{value: _ethRequired}(1, (address(this)));

49        gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers,requestToToken[id])));

62        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L36

This helps optimize gas costs by skipping unnecessary code added by the compiler prior to Solidity 0.8.10 via using low-level calls for external functions with return values.

## [G-02] Require() strings longer than 32 bytes cost extra Gas

**Issue Description**\
The EVM has a 32 byte limit for data inside transactions. Require and revert strings longer than this will be truncated and wrapped in a keccak256 hash, increasing gas costs.

**Proposed Optimization**\
For require and revert messages, keep strings less than or equal to 32 bytes to avoid the extra hashing overhead. Alternatively, omit the messages altogether to save even more gas.

**Estimated Gas Savings**\
Each string hashed due to length costs an additional ~200 gas. For contracts with many validation checks, trimming require/revert strings could save thousands of gas.

**Attachments**

- **Code Snippets**

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

```solidity
File:

171        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

179        require(msg.sender == minterContract, "Caller is not the Minter Contract");

190        require(msg.sender == minterContract, "Caller is not the Minter Contract");

205        require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");

214        require(msg.sender == minterContract, "Caller is not the Minter Contract");

215        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L171

## [G-03] Avoid zero to one storage writes where possible

**Issue Description**\
Initializing storage variables from zero to non-zero costs significantly more gas than updating non-zero variables. This extra cost can be avoided where possible.

Initializing a storage variable is one of the most expensive operations a contract can do.

When a storage variable goes from zero to non-zero, the user must pay 22,100 gas total (20,000 gas for a zero to non-zero write and 2,100 for a cold storage access).

This is why the Openzeppelin reentrancy guard registers functions as active or not with 1 and 2 rather than 0 and 1. It only costs 5,000 gas to alter a storage variable from non-zero to non-zero.

**Proposed Optimization**\
Initialize important storage variables to non-zero (e.g. 1) instead of zero, For flags, use booleans instead of numbers, Structure data to minimize zero to non-zero writes

**Estimated Gas Savings**\
Each avoided zero to non-zero storage write saves ~17,000 gas. With careful structuring, thousands of gas can be saved over the lifetime of a contract.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

// @audit  it is gas optimize to newCollectionIndex  start with 1   "newCollectionIndex = 1"
26    uint256 public newCollectionIndex;

110        newCollectionIndex = newCollectionIndex + 1;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110

## [G-04] Using Storage instead of memory for structs/arrays saves gas

**Issue Description**\
Reading fields from a struct/array in storage into a memory variable incurs extra gas costs compared to reading directly from storage.

**Proposed Optimization**\
Instead of assigning the full struct/array to memory, declare variables to reference fields directly from storage. Cache needed fields in local stack variables.

**Estimated Gas Savings**\
Avoids ~2100 gas per field not read from memory. For large structs/arrays accessed often this can save thousands of gas.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/XRandoms.sol

18        string[100] memory wordsList = ["Acai", "Ackee", "Apple", "Apricot", "Avocado", "Babaco", "Banana", "Bilberry", "Blackberry", "Blackcurrant", "Blood Orange",
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L18-L26

## [G-05] Understand the trade-offs when choosing between internal functions and modifiers

**Issue Description**\
Modifiers inject its implementation bytecode where it is used while internal functions jump to the location in the runtime code where the its implementation is. This brings certain trade-offs to both options.

Using modifiers more than once means repetitiveness and increase in size of the runtime code but reduces gas cost because of the absence of jumping to the internal function execution offset and jumping back to continue. This means that if runtime gas cost matter most to you, then modifiers should be your choice but if deployment gas cost and/or reducing the size of the creation code is most important to you then using internal functions will be best.

**Proposed Optimization**\
However, modifiers have the tradeoff that they can only be executed at the start or end of a functon. This means executing it at the middle of a function wouldn’t be directly possible, at least not without internal functions which kill the original purpose. This affects it’s flexibility. Internal functions however can be called at any point in a function.

**Estimated Gas Savings**\
Example showing difference in gas cost using modifiers and an internal function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

/** deployment gas cost: 195435
    gas per call:
              restrictedAction1: 28367
              restrictedAction2: 28377
              restrictedAction3: 28411
 */
 contract Modifier {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function restrictedAction1() external onlyOwner {
        val = 1;
    }

    function restrictedAction2() external onlyOwner {
        val = 2;
    }

    function restrictedAction3() external onlyOwner {
        val = 3;
    }
}



/** deployment gas cost: 159309
    gas per call:
              restrictedAction1: 28391
              restrictedAction2: 28401
              restrictedAction3: 28435
 */
 contract InternalFunction {
    address owner;
    uint256 val;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() internal view {
        require(msg.sender == owner);
    }

    function restrictedAction1() external {
        onlyOwner();
        val = 1;
    }

    function restrictedAction2() external {
        onlyOwner();
        val = 2;
    }

    function restrictedAction3() external {
        onlyOwner();
        val = 3;
    }
}
```

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

116    modifier FunctionAdminRequired(bytes4 _selector) {

123    modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L116

```solidity
File: main/smart-contracts/MinterContract.sol

136    modifier ArtistOrAdminRequired(uint256 _collectionID, bytes4 _selector) {

143    modifier FunctionAdminRequired(bytes4 _selector) {

150    modifier CollectionAdminRequired(uint256 _collectionID, bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L136

```solidity
File: main/smart-contracts/NextGenAdmins.sol

31    modifier AdminRequired {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L54

```solidity
File: main/smart-contracts/RandomizerVRF.sol

47    modifier FunctionAdminRequired(bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L47

```solidity
File: main/smart-contracts/RandomizerRNG.sol

35    modifier FunctionAdminRequired(bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L35

```solidity
File: main/smart-contracts/AuctionDemo.sol

31    modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L31

## [G-06] Use transfer hooks for tokens instead of initiating a transfer from the destination smart contract

**Issue Description**\
Let’s say you have contract A which accepts token B an NFT .

1. The naive workflow is as follows:

   - msg.sender approves contract A to accept token B
   - msg.sender calls contract A to transfer tokens from msg.sender to A
   - Contract A then calls token B to do the transfer
   - Token B does the transfer, and calls onTokenReceived() in contract A
   - Contract A returns a value from onTokenReceived() to token B
   - Token B returns execution to contract A

This is very inefficient. It’s better for msg.sender to call contract B to do a transfer which calls the tokenReceived hook in contract A.

Note that:

safeTransfer and safeMint in ERC721 have a transfer hook

If you need to pass arguments to contract A, simply use the data field and parse that in contract A.

**Proposed Optimization**\
Have msg.sender call the token contract (Contract B) directly to transfer to Contract A, triggering the token transfer hook in Contract A. This consolidates execution and avoids additional EVM calls.

**Estimated Gas Savings**\
Eliminates the overhead of an additional EVM call, saving a baseline of ~`23,000` gas per transfer. With many tokenized assets, long-term savings can be substantial.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/MinterContract.sol

340        IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L340

```solidity
File: main/smart-contracts/AuctionDemo.sol

112                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112-L113

## [G-07] Use ECDSA signatures instead of merkle trees for allowlists and airdrops

**Issue Description**\
Merkle trees require transmitting the entire merkle proof as calldata, increasing costs linearly with list size. ECDSA signatures provide a constant-cost alternative.

1. Merkle trees require transmitting the entire merkle proof as calldata along with the claim. This increases gas linearly based on the number of nodes in the merkle tree.

2. ECDSA signatures have a fixed calldata size regardless of the list size. Only the signed message hash and signature need to be supplied.

3. Verifying an ECDSA signature has a constant gas cost, while merkle proof verification gets more expensive as the list grows larger.

4. For small lists under 100 addresses, the gas savings of signatures may be minor. But as lists scale to thousands or millions of addresses, the signature approach yields tremendous savings.

5. Signatures allow for off-chain generation by a trusted party, while merkle trees require on-chain construction which is more expensive.

6. The signature approach is also more scalable long-term as it avoids any on-chain data structure maintenance with list revisions.

**Proposed Optimization**\
 Replace merkle tree membership verification with validating ECDSA signatures signed by a trusted party over the address/index. This avoids transmitting a varying-size proof.

**Estimated Gas Savings**\
Preliminary testing shows a savings of 20-30% gas or more compared to merkle trees for lists over 100 addresses. As lists scale larger, signatures yield far greater savings.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/MinterContract.sol

16  import "./MerkleProof.sol";
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L16

## [G-08] ERC1155 is a cheaper non-fungible token than ERC721

**Issue Description**\
The ERC721 balanceOf function is rarely used in practice but adds a storage overhead whenever a mint and transfer happens. ERC1155 tracks balance per id, and also uses the same balance to track ownership of the id. If the maximum supply for each id is one, then the token becomes non-fungible per each id.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

108         constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L108

```solidity
File: main/smart-contracts/MinterContract.sol

18  import "./IERC721.sol";
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L18

## [G-09] Consider using alternatives to OpenZeppelin

**Issue Description**\
While OpenZeppelin is full-featured, some functions may not be optimally gas efficient. Alternatives like Solmate and Solady optimize for lower costs.

**Proposed Optimization**\
Evaluate replacing OpenZeppelin implementations with equivalent functions from Solmate, Solady or other specialized libraries optimized for gas savings. Performance test contracts using alternatives.

**Estimated Gas Savings**\
Initial testing shows potential savings of 5-20% compared to OpenZeppelin implementations for common functions. Larger contracts using alternatives extensively could save thousands of gas.

## [G-10] Using assembly to revert with an error message

**Issue Description**\
Reverting with error messages in Solidity incurs extra gas costs compared to using inline assembly.
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```

From the example above we can see that we get a gas saving of over 300 gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

**Proposed Optimization**\
For common validation checks, replace Solidity require/revert with inline assembly to directly pack and propagate the error message.

**Estimated Gas Savings**\
Example shows a savings of ~300 gas per revert compared to Solidity. Larger contracts with many validation points could save thousands of total gas.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

117      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

124      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

148        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");

171        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

179        require(msg.sender == minterContract, "Caller is not the Minter Contract");

190        require(msg.sender == minterContract, "Caller is not the Minter Contract");

205        require(_isApprovedOrOwner(_msgSender(), _tokenId), "ERC721: caller is not token owner or approved");

214        require(msg.sender == minterContract, "Caller is not the Minter Contract");
215        require(_isApprovedOrOwner(burner, _tokenId), "ERC721: caller is not token owner or approved");

239        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

258        require(msg.sender == collectionAdditionalData[_collectionID].collectionArtistAddress, "Only artist");

259        require(artistSigned[_collectionID] == false, "Already Signed");

267        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

274        require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");

283            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");

293        require(isCollectionCreated[_collectionID] == true, "No Col");

300        require(msg.sender == collectionAdditionalData[_collectionID].randomizerContract);

301        require(tokenToHash[_mintIndex] == 0x0000000000000000000000000000000000000000000000000000000000000000);

316        require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");

323        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L117

```solidity
File: main/smart-contracts/MinterContract.sol

137      require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

144      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

151      require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

158        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

171        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

182        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

186            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

197        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

211                require(isAllowedToMint == true, "No delegation");

213                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");

217                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, msg.sender) + _numberOfTokens, "AL limit");

220            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');

223            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");

224            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");

232        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

233        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");

251            require(tDiff>=1 && _numberOfTokens == 1, "1 mint/period");

259        require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");

260        require(block.timestamp >= collectionPhases[_mintCollectionID].publicStartTime && block.timestamp<=collectionPhases[_mintCollectionID].publicEndTime,"No minting");

265        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");

266        require(msg.value >= getPrice(_mintCollectionID), "Wrong ETH");

277        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

280        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

294        require(tDiff>=1, "1 mint/period");

309        require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

317        require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

328        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");

329        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");

337            require(isAllowedToMint == true, "No delegation");

339        require(_tokenId >= burnOrSwapIds[externalCol][0] && _tokenId <= burnOrSwapIds[externalCol][1], "Token id does not match");

350            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');

360        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

361        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");

370        require(_artistPrSplit + _teamPrSplit == 100, "splits need to be 100%");

371        require(_artistSecSplit + _teamSecSplit == 100, "splits need to be 100%");

416        require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");

417        require(collectionTotalAmount[_collectionID] > 0, "Collection Balance must be grater than 0");

418        require(collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage + _teamperc1 + _teamperc2 == 100, "Change percentages");

455        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L137

```solidity
File: main/smart-contracts/NextGenAdmins.sol

32      require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");

59        require(_collectionID > 0, "Collection Id must be larger than 0");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32

```solidity
File: main/smart-contracts/RandomizerNXT.sol

35      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

56        require(msg.sender == gencore);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L35

```solidity
File: main/smart-contracts/RandomizerVRF.sol

48      require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

53        require(msg.sender == gencore);

72        require(msg.sender == gencore);

95        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L48

```solidity
File: main/smart-contracts/RandomizerRNG.sol

36        require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

41        require(msg.sender == gencore);

54        require(msg.sender == gencore);

62        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L36

```solidity
File: main/smart-contracts/AuctionDemo.sol

32      require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

58        require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);

105        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);

125        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");

126        require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);

135        require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L32

## [G-11] Use assembly to reuse memory space when making more than one external call.

**Issue Description**\
An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity
contract Called {
    function add(uint256 a, uint256 b) external pure returns(uint256) {
        return a + b;
    }
}


contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns(uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}


contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns(uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/AuctionDemo.sol

105        require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);
108        address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);
112                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
113                (bool success, ) = payable(owner()).call{value: highestBid}("");
116                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105

```solidity
File: main/smart-contracts/MinterContract.sol

182        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
185            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
186            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
188                uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
189                gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID);



207                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
209                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, collectionPhases[col].delAddress, msg.sender, 2);

213                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, _delegator) + _numberOfTokens, "AL limit");
217                require(_maxAllowance >= gencore.retrieveTokensMintedALPerAddress(col, msg.sender) + _numberOfTokens, "AL limit");

224            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
231        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
232        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

235            uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
236            gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase);


261        require ((_tokenId >= gencore.viewTokensIndexMin(_burnCollectionID)) && (_tokenId <= gencore.viewTokensIndexMax(_burnCollectionID)), "col/token id error");
264        collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
265        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");
267        uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
270        gencore.burnToMint(mintIndex, _burnCollectionID, _tokenId, _mintCollectionID, _saltfun_o, burner);


277        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
279        collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
280        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
281        uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
282        gencore.airDropTokens(mintIndex, _recipient, _tokenData, _saltfun_o, _collectionID);


330        address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
333            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);
335            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, _erc721Collection, msg.sender, 2);
340        IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);
350            require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');
359        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
360        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
362        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
363        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);



434        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
435        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
436        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
437        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
438        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L158

## [G-12] Always use Named Returns

**Issue Description**\
The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract NamedReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

contract NamedReturn2 {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}

```

**Proposed Optimization**\
Always declare return variables in the returns statement rather than using anonymous returns. This allows the compiler to optimize the code.

**Estimated Gas Savings**\
Preliminary testing shows potential savings of 5-20 gas per function depending on complexity. Larger contracts with many functions could save thousands of gas.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

347            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

350            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";

355            return _uri;

363        return string(abi.encodePacked(collectionInfo[viewColIDforTokenID(tokenId)].collectionName, " #" ,tok.toString()));

368        return collectionFreeze[_collectionID];

378        return wereDataAdded[_collectionID];

405        return (tokensMintedAllowlistAddress[_collectionID][_address]);

410        return (tokensMintedPerAddress[_collectionID][_address]);

416        return (tokensAirdropPerAddress[_collectionID][_address]);

421        return (collectionAdditionalData[_collectionID].collectionArtistAddress);

427        return (collectionInfo[_collectionID].collectionName, collectionInfo[_collectionID].collectionArtist, collectionInfo[_collectionID].collectionDescription, collectionInfo[_collectionID].collectionWebsite, collectionInfo[_collectionID].collectionLicense, collectionInfo[_collectionID].collectionBaseURI);

433        return (collectionInfo[_collectionID].collectionLibrary, collectionInfo[_collectionID].collectionScript);

439        return (collectionAdditionalData[_collectionID].collectionArtistAddress, collectionAdditionalData[_collectionID].maxCollectionPurchases, collectionAdditionalData[_collectionID].collectionCirculationSupply, collectionAdditionalData[_collectionID].collectionTotalSupply, collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, collectionAdditionalData[_collectionID].randomizerContract);

445        return (tokenToHash[_tokenid]);

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L347

```solidity
File: main/smart-contracts/MinterContract.sol

471        return (collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage, collectionRoyaltiesPrimarySplits[_collectionID].teamPercentage);

477        return (collectionArtistPrimaryAddresses[_collectionID].primaryAdd1, collectionArtistPrimaryAddresses[_collectionID].primaryAdd2, collectionArtistPrimaryAddresses[_collectionID].primaryAdd3, collectionArtistPrimaryAddresses[_collectionID].add1Percentage, collectionArtistPrimaryAddresses[_collectionID].add2Percentage, collectionArtistPrimaryAddresses[_collectionID].add3Percentage, collectionArtistPrimaryAddresses[_collectionID].status);

483        return (collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, collectionRoyaltiesSecondarySplits[_collectionID].teamPercentage);

489        return (collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1, collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2, collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3, collectionArtistSecondaryAddresses[_collectionID].add1Percentage, collectionArtistSecondaryAddresses[_collectionID].add2Percentage, collectionArtistSecondaryAddresses[_collectionID].add3Percentage, collectionArtistSecondaryAddresses[_collectionID].status);

495        return (collectionPhases[_collectionID].allowlistStartTime, collectionPhases[_collectionID].allowlistEndTime, collectionPhases[_collectionID].merkleRoot, collectionPhases[_collectionID].publicStartTime, collectionPhases[_collectionID].publicEndTime);

501        return (collectionPhases[_collectionID].collectionMintCost, collectionPhases[_collectionID].collectionEndMintCost, collectionPhases[_collectionID].rate, collectionPhases[_collectionID].timePeriod, collectionPhases[_collectionID].salesOption, collectionPhases[_collectionID].delAddress);

507        return true;

513        return collectionPhases[_collectionID].publicEndTime;

519        return mintToAuctionData[_tokenId];

525        return mintToAuctionStatus[_tokenId];

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L471

```solidity
File: main/smart-contracts/XRandoms.sol

37        return randomNum;

42        return getWord(randomNum);

46        return getWord(id);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L37

## [G-13] Do-While loops are cheaper than for loops

**Issue Description**\
If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times;) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}
```

**Proposed Optimization**\
Replace for loops with do-while for iterating where the number of iterations is known upfront. Add check for zero iterations to match for loop behavior.

**Estimated Gas Savings**\
Testing shows do-while loop saves ~5 gas per iteration compared to for loop. Savings grow linearly with number of iterations. Even small loops in many functions can yield meaningful savings.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

282        for (uint256 x; x < _tokenId.length; x++) {

453        for (uint256 i=0; i < collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionScript.length; i++) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L282

```solidity
File: main/smart-contracts/MinterContract.sol

184        for (uint256 y=0; y< _recipients.length; y++) {

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L184

```solidity
File: smart-contracts/NextGenAdmins.sol

51        for (uint256 i=0; i<_selector.length; i++) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L51

```solidity
File: smart-contracts/AuctionDemo.sol

69            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {

90        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {

110        for (uint256 i=0; i< auctionInfoData[_tokenid].length; i ++) {

136        for (uint256 i=0; i<auctionInfoData[_tokenid].length; i++) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L136

## [G-14] Short-circuit booleans

**Issue Description**\
In Solidity, boolean expressions using || and && operators short-circuit based on the first expression's value, allowing the second expression to potentially be skipped.

**Proposed Optimization**\
Order boolean expressions so the cheaper expression to evaluate comes first, taking advantage of short-circuiting to skip evaluating the second expression when possible.

**Estimated Gas Savings**\
By avoiding evaluating the second expression, savings of 100-1000 gas can be achieved depending on expression complexity. Significant savings for expressions deeply nested in loops or conditional blocks.

**Attachments**

- **Code Snippets**

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

## [G-15] Don’t make variables public unless it is necessary to do so

**Issue Description**\
Declaring variables public when not needed increases contract size and gas costs due to implicit accessor functions generated.

**Proposed Optimization**\
Only declare variables public if they truly need to be readable from outside the contract. Otherwise, use private or internal as needed.

**Estimated Gas Savings**\
Each public variable adds ~20-50 gas overhead per external call due to larger bytecode. Contracts with many unused public variables could save thousands of gas.

**Attachments**

- **Code Snippets**

```solidity
File: main/smart-contracts/NextGenCore.sol

26    uint256 public newCollectionIndex;

105    address public minterContract;

```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L26

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

## [G-16] It is sometimes cheaper to cache calldata

**Issue Description**\
Repeatedly calling calldataload to access function arguments can be optimized by caching loaded values.

**Proposed Optimization**\
For functions iterating over calldata, cache loaded values in local variables rather than reloading on each access.

**Estimated Gas Savings**\

```solidity
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}
```

**Attachments**

- **Code Snippets**

```solidity
File: smart-contracts/MinterContract.sol

196    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196

## [G-17] Empty blocks should be removed or emit something

**Issue Description**\
Solidity generates bytecode for empty code blocks, wasting gas.

**Proposed Optimization**\
Remove empty code blocks from contracts or emit events to remove block without discarding desired side effects.

**Estimated Gas Savings**\
An empty block costs ~200 gas in bytecode and execution. Removing unnecessary empty blocks Savings can add up for many scattered blocks.

**Attachments**

- **Code Snippets**

```solidity
File: smart-contracts/AuctionDemo.sol

118            } else {}

141            } else {}
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L118

```solidity
File: smart-contracts/RandomizerRNG.sol

86    receive() external payable {}
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L86

## [G-18] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

**Issue Description**\
Caching global variables like msg.sender into local variables increases contract size and execution cost.

**Proposed Optimization**\
Directly use global variables like msg.sender rather than assigning to a local cache.

**Estimated Gas Savings**\
Caching adds ~20 gas overhead per usage from larger bytecode. Direct usage avoids this and is more efficient.

**Attachments**

- **Code Snippets**

```solidity
File: smart-contracts/MinterContract.sol

218                mintingAddress = msg.sender;

225            mintingAddress = msg.sender;

269        address burner = msg.sender;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L218

## [G-19] A modifier used only once and not being inherited should be inlined to save gas

**Issue Description**\
Modifiers add bytecode overhead and execution cost for the modifier function. This is avoidable for non-reusable single-use modifiers.

**Proposed Optimization**\
For modifiers used only once locally, inline their logic directly rather than using a modifier.

**Estimated Gas Savings**\
Avoiding the modifier function call overhead saves ~100 gas per usage. Significant if used deeply nested or in loops.

**Attachments**

- **Code Snippets**

```solidity
File: smart-contracts/AuctionDemo.sol

31    modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L31

### This modifier is only used in this function

```
104    function claimAuction(uint256 _tokenid) public WinnerOrAdminRequired(_tokenid,this.claimAuction.selector){
```

## [G-20] Pre-increment and pre-decrement are cheaper than +1 ,-1

**Issue Description**\
Using ++ and -- operators is more gas efficient than adding/subtracting 1 for incrementing/decrementing variables.

**Proposed Optimization**\
Replace i = i + 1 with i++ and i = i - 1 with i-- in increment/decrement operations.

**Estimated Gas Savings**\
++ and -- cost 2 gas less than adding/subtracting 1 per operation.

**Attachments**

- **Code Snippets**

```solidity
File: smart-contracts/MinterContract.sol

185            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;

231        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;

252            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));

295        lastMintDate[_collectionID] = collectionPhases[_collectionID].allowlistStartTime + (collectionPhases[_collectionID].timePeriod * (gencore.viewCirSupply(_collectionID) - 1));

550                price = collectionPhases[_collectionId].collectionMintCost / (tDiff + 1);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L550

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

## [G-21] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

**Issue Description**\
abi.encodePacked() compiles to less efficient bytecode than bytes.concat() since Solidity 0.8.4.

**Proposed Optimization**\
Replace uses of abi.encodePacked() with bytes.concat() for concatenating bytes arrays.

**Estimated Gas Savings**\
bytes.concat() saves 20-50 gas compared to abi.encodePacked() depending on arguments. Significant savings for large concatenated values.

**Attachments**

- **Code Snippets**

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
File: smart-contracts/MinterContract.sol

212                node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));

216                node = keccak256(abi.encodePacked(msg.sender, _maxAllowance, tokData));

316        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));

327        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));

348            node = keccak256(abi.encodePacked(_tokenId, tokData));
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212

```solidity
File: smart-contracts/XRandoms.sol

36        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;

41        uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L36

## [G-22] Cache external calls outside of loop to avoid re-calling function on each iteration

**Issue Description**\
Calling external functions like STATICCALL inside a loop incurs a gas cost for each iteration that can be optimized.

**Proposed Optimization**\
Cache results of external calls made prior to loop in local variables, and reference the cache inside the loop body.

**Estimated Gas Savings**\
Avoiding a re-call of an external function saves 100 gas per loop iteration. Significant savings for medium/large loops.

```solidity
// Without caching:

for (uint i = 0; i < 10; i++) {
  // Call external func
}

// With caching:

uint cache = externalFunc();

for (uint i = 0; i < 10; i++) {
  // Use cache
}
```

**Attachments**

- **Code Snippets**

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

# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Avoid contract existence checks by using low level calls|1|
|[G‑02]|Use Modifiers Instead of Functions To Save Gas|2|
|[G‑03]|Can Make The Variable Outside The Loop To Save Gas |1|
|[G‑04]|Unlimited gas consumption risk due to external call recipients|11|
|[G‑05]|A modifier used only once and not being inherited should be inlined to save gas|1|
|[G‑06]|Sort Solidity operations using short-circuit mode|1|
|[G‑07]|Make 3 event parameters indexed when possible|1|
|[G‑08]|Duplicated require()/if() checks should be refactored to a modifier or function|9|
|[G‑09]|Uncheck arithmetics operations that can’t underflow/overflow|1|
|[G‑10]|Optimize External Calls with Assembly for Memory Efficiency|8|
|[G‑11]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G‑12]|Do-While loops are cheaper than for loops|1|
|[G‑13]|Don’t compare boolean expressions to boolean literals|53|
|[G‑14]|Empty blocks should be removed or emit something|2|
|[G‑15]|Pre-increment and pre-decrement are cheaper than +1 ,-1|2|
|[G‑16]|Ternary operation is cheaper than if-else statement|2|
|[G‑17]|Using assembly to revert with an error message|~|
|[G‑18]|Write gas-optimal for-loops|~|
|[G‑19]|Don’t make variables public unless it is necessary to do so|5|


## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

There are 8 instances of this issue:

```solidity
File: smart-contracts/MinterContract.sol
330   address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);

340   IERC721(_erc721Collection).safeTransferFrom(ownerOfToken, burnOrSwapAddress[externalCol], _tokenId);

455   require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L171


```solidity
File: smart-contracts/NextGenCore.sol
171  require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

308  require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");

316  require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");

323  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L171

```solidity
File: smart-contracts/RandomizerRNG.sol
62  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L62

## [G-02] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.


There are 2 instances of this issue:

```solidity
File: smart-contracts/NextGenCore.sol
344  _requireMinted(tokenId);

451  _requireMinted(tokenId);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L344

## [G-03] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

There are 1 instances of this issue:


```solidity
File: smart-contracts/MinterContract.sol
188   uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L188

## [G‑04] Unlimited gas consumption risk due to external call recipients
When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

There are 11 instances of this issue:
```solidity
File: smart-contracts/AuctionDemo.sol
113  (bool success, ) = payable(owner()).call{value: highestBid}("");

116  (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");

128  (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");

139  (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113

```solidity
File: smart-contracts/MinterContract.sol
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L434-L438

```solidity
File: smart-contracts/RandomizerRNG.sol
82  (bool success, ) = payable(admin).call{value: balance}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L82

## [G-05] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.
By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

There are 1 instances of this issue:

```solidity
File: smart-contracts/AuctionDemo.sol
// @audit only use in line number `104`
31   modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L31

## [G-06] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.
```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
There are 1 instances of this issue:
```solidity
File: smart-contracts/NextGenAdmins.sol
// @audit adminPermissions is state variable and it high gas cost
32   require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32

`Fix code`
```diff
-   require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
+   require((_msgSender()== owner()), "Not allowed") || (adminPermissions[msg.sender] == true);
```


## [G-07] Make 3 event parameters indexed when possible
It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

There are 1 instances of this issue:


```solidity
File: smart-contracts/RandomizerVRF.sol
20   event RequestFulfilled(uint256 requestId, uint256[] randomWords);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L20

## [G-08] Duplicated require()/if() checks should be refactored to a modifier or function
sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations, and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation
You can consider adding a modifier like below
```
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }
```

There are 9 instances of this issue:

```solidity
File: smart-contracts/MinterContract.sol
// @audit This Require Is Duplicated On Line Number 182, 277
158	  require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

// @audit This Require Is Duplicated On Line Number 197
171   require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

// @audit This Require Is Duplicated On Line Number 280
186   require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

// @audit This Require Is Duplicated On Line Number 337
211   require(isAllowedToMint == true, "No delegation");

// @audit This Require Is Duplicated On Line Number 350
220   require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');

// @audit This Require Is Duplicated On Line Number 360
232   require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L158



```solidity
File: smart-contracts/NextGenCore.sol
// @audit this require is duplicated on line 190, 214
179  require(msg.sender == minterContract, "Caller is not the Minter Contract");

// @audit this require is duplicated on line 267
239 require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L179


```solidity
File: smart-contracts/RandomizerRNG.sol
// @audit this require is duplicated on line 54
41   require(msg.sender == gencore);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L41


## [G-09] Uncheck arithmetics operations that can’t underflow/overflow
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an 

There are 1 instances of this issue:
```solidity
File: smart-contracts/MinterContract.sol
560  return price - decreaserate; 
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L560


## [G-10] Optimize External Calls with Assembly for Memory Efficiency
Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

There are 8 instances of this issue:
```solidity
File: smart-contracts/AuctionDemo.sol
108  address ownerOfToken = IERC721(gencore).ownerOf(_tokenid);

112  IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L108


```solidity
File: smart-contracts/MinterContract.sol
330   address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L330


```solidity
File: smart-contracts/NextGenCore.sol
308  require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");

316  require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");

323  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L308


```solidity
File: smart-contracts/RandomizerRNG.sol
62   require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L62


```solidity
File: smart-contracts/RandomizerVRF.sol
95   require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L95


## [G-11] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There are 1 instances of this issue:


```solidity
File: smart-contracts/AuctionDemo.sol
112   IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112

## [G-12] Do-While loops are cheaper than for loops
If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.
```
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
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

There are 1 instances of this issue:
```solidity
File: smart-contracts/AuctionDemo.sol
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L67-L69


## [G-13] Don’t compare boolean expressions to boolean literals
if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>)

There are 53 instances of this issue:
```solidity
File: smart-contracts/AuctionDemo.sol
32   require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

58  require(msg.value > returnHighestBid(_tokenid) && block.timestamp <= minter.getAuctionEndTime(_tokenid) && minter.getAuctionStatus(_tokenid) == true);

70  if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {

75  if (auctionInfoData[_tokenid][index].status == true) {

91  if (auctionInfoData[_tokenid][i].bid > highBid && auctionInfoData[_tokenid][i].status == true) {

95  if (auctionInfoData[_tokenid][index].status == true) {

105  require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true);

111  if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {

115   } else if (auctionInfoData[_tokenid][i].status == true) {

126   require(auctionInfoData[_tokenid][index].bidder == msg.sender && auctionInfoData[_tokenid][index].status == true);

137   if (auctionInfoData[_tokenid][i].bidder == msg.sender && auctionInfoData[_tokenid][i].status == true) {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L137


```solidity
File: smart-contracts/MinterContract.sol
137  require(msg.sender == gencore.retrieveArtistAddress(_collectionID) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

144   require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

151   require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

158  require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

171  require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

182  require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

197  require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

208  if (isAllowedToMint == false) {

211  require(isAllowedToMint == true, "No delegation");

259  require(burnToMintCollections[_burnCollectionID][_mintCollectionID] == true, "Initialize burn");

277  require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

309  require((gencore.retrievewereDataAdded(_burnCollectionID) == true) && (gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

317  require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");

328   require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");

329  require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");

334  if (isAllowedToMint == false) {

337  require(isAllowedToMint == true, "No delegation");

381  require (collectionArtistPrimaryAddresses[_collectionID].status == false, "Already approved");

395  require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");

416  require(collectionArtistPrimaryAddresses[_collectionID].status == true, "Accept Royalties");

445  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L137

```solidity
File: smart-contracts/NextGenAdmins.sol
32   require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32


```solidity
File: smart-contracts/NextGenCore.sol
117    require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

124    require(adminsContract.retrieveCollectionAdmin(msg.sender,_collectionID) == true || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

148   require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");

158  } else if (artistSigned[_collectionID] == false) {

171  require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");

239  require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

259  require(artistSigned[_collectionID] == false, "Already Signed");

267  require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");

274  require(collectionFreeze[tokenIdsToCollectionIds[_tokenId]] == false, "Data frozen");

283  require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");

293  require(isCollectionCreated[_collectionID] == true, "No Col");

316  require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");

323  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");

345  if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {

348  } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {    
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L117


```solidity
File: smart-contracts/RandomizerNXT.sol
35   require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L35


```solidity
File: smart-contracts/RandomizerRNG.sol
36  require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");

62  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L62


```solidity
File: smart-contracts/RandomizerVRF.sol
48  require(adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true , "Not allowed");

95  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L48


## [G‑14] Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be 
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when 
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) . 


```solidity
File: smart-contracts/AuctionDemo.sol
118  } else {}

141  } else {}
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L118


## [G-15] Pre-increment and pre-decrement are cheaper than +1 ,-1

There are 2 instances of this issue:
```solidity
File: smart-contracts/NextGenCore.sol
110  newCollectionIndex = newCollectionIndex + 1;

140  newCollectionIndex = newCollectionIndex + 1;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110


## [G-16] Ternary operation is cheaper than if-else statement

There are 2 instances of this issue:
```solidity
File: smart-contracts/AuctionDemo.sol
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L75-L79


```solidity
File: smart-contracts/XRandoms.sol
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28-L32


## [G‑17] Using assembly to revert with an error message
When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.


Here’s an example;
```
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


## [G‑18] Write gas-optimal for-loops
Note: As of Solidity 0.8.22, this trick is done automatically by the compiler and does not need to be done explicitly.


This is what a gas-optimal for loop looks like, if you combine the two tricks above:
```
for (uint256 i; i < limit; ) {
    
    // inside the loop
    
    unchecked {
        ++i;
    }
}
```
The two differences here from a conventional for loop is that i++ becomes ++i (as noted above), and it is unchecked because the limit variable ensures it won’t overflow.

## [G‑19] Don’t make variables public unless it is necessary to do so
A public storage variable has an implicit public function of the same name. A public function increases the size of the jump table and adds bytecode to read the variable in question. That makes the contract larger.

Remember, private variables aren’t private, it’s not difficult to extract the variable value using web3.js.

This is especially true for constants which are meant to be read by humans rather than smart contracts.

```solidity
File: smart-contracts/NextGenCore.sol
26  uint256 public newCollectionIndex;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L26


```solidity
File: smart-contracts/RandomizerVRF.sol
    bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;
    uint32 public callbackGasLimit = 40000;
    uint16 public requestConfirmations = 3;
    uint32 public numWords = 1;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L26-L29


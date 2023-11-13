# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |  instance  |
|------|---------|------------|
|[G‑01]|Unlimited gas consumption risk due to external call recipients|11|
|[G‑02]|Duplicated require()/if() checks should be refactored to a modifier or function|9|
|[G‑03]|Optimize External Calls with Assembly for Memory Efficiency|8|
|[G‑04]|Empty blocks should be removed or emit something|2|
|[G‑05]|Pre-increment and pre-decrement are cheaper than +1 ,-1|2|
|[G‑06]|Ternary operation is cheaper than if-else statement|2|
|[G‑07]|Avoid contract existence checks by using low level calls|8|
|[G‑08]|Use Modifiers Instead of Functions To Save Gas|2|
|[G‑09]|Uncheck arithmetics operations that can’t underflow/overflow|1|
|[G‑10]|Cache external calls outside of loop to avoid re-calling function on each iteration|1|
|[G‑11]|Do-While loops are cheaper than for loops|1|
|[G‑12]|Can Make The Variable Outside The Loop To Save Gas |1|
|[G‑13]|A modifier used only once and not being inherited should be inlined to save gas|1|
|[G‑14]| Sort Solidity operations using short-circuit mode|1|
|[G‑15]| Make 3 event parameters indexed when possible|1|



## [G‑01] Unlimited gas consumption risk due to external call recipients

When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

```solidity
File: smart-contracts/RandomizerRNG.sol
82  (bool success, ) = payable(admin).call{value: balance}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L82


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



## [G-02] Duplicated require()/if() checks should be refactored to a modifier or function

sing modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. 




```solidity
File: smart-contracts/NextGenCore.sol
179  require(msg.sender == minterContract, "Caller is not the Minter Contract");

239 require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L179


```solidity
File: smart-contracts/RandomizerRNG.sol
41   require(msg.sender == gencore);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L41

```solidity
File: smart-contracts/MinterContract.sol
158	  require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

171   require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

186   require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

211   require(isAllowedToMint == true, "No delegation");

220   require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');

232   require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L158






## [G-03] Optimize External Calls with Assembly for Memory Efficiency

Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.
Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

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




## [G‑04] Empty blocks should be removed or emit something


```solidity
File: smart-contracts/AuctionDemo.sol
118  } else {}

141  } else {}
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L118


## [G-05] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: smart-contracts/NextGenCore.sol
110  newCollectionIndex = newCollectionIndex + 1;

140  newCollectionIndex = newCollectionIndex + 1;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L110


## [G-06] Ternary operation is cheaper than if-else statement



```solidity
File: smart-contracts/XRandoms.sol
        if (id==0) {
            return wordsList[id];
        } else {
            return wordsList[id - 1];
        }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L28-L32

```solidity
File: smart-contracts/AuctionDemo.sol
            if (auctionInfoData[_tokenid][index].status == true) {
                return highBid;
            } else {
                return 0;
            }
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L75-L79




## [G‑07] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.



```solidity
File: smart-contracts/RandomizerRNG.sol
62  require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L62


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

## [G-08] Use Modifiers Instead of Functions To Save Gas




```solidity
File: smart-contracts/NextGenCore.sol
344  _requireMinted(tokenId);

451  _requireMinted(tokenId);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L344



## [G-09] Uncheck arithmetics operations that can’t underflow/overflow

When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an 

```solidity
File: smart-contracts/MinterContract.sol
560  return price - decreaserate; 
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L560





## [G-10] Cache external calls outside of loop to avoid re-calling function on each iteration

In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.



```solidity
File: smart-contracts/AuctionDemo.sol
112   IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112

## [G-11] Do-While loops are cheaper than for loops

Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.


```solidity
File: smart-contracts/AuctionDemo.sol
        if (auctionInfoData[_tokenid].length > 0) {
            uint256 highBid = 0;
            for (uint256 i=0; i< auctionInfoData[_tokenid].length; i++) {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L67-L69





## [G-12] Can Make The Variable Outside The Loop To Save Gas 

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract.




```solidity
File: smart-contracts/MinterContract.sol
188   uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L188



## [G-13] A modifier used only once and not being inherited should be inlined to save gas


By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.


```solidity
File: smart-contracts/AuctionDemo.sol
// @audit only use in line number `104`
31   modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L31




## [G-14] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back.

```solidity
File: smart-contracts/NextGenAdmins.sol
32   require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32



## [G-15] Make 3 event parameters indexed when possible

It is the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.



```solidity
File: smart-contracts/RandomizerVRF.sol
20   event RequestFulfilled(uint256 requestId, uint256[] randomWords);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L20

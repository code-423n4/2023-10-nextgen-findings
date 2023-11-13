

### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Cache External Calls in Loops | [Cache External Calls in Loops](#cache-external-calls-in-loops) | 3 | 300 |
|[G-2] Short-circuit rules can be used to optimize some gas usage | [Short-circuit rules can be used to optimize some gas usage](#short-circuit-rules-can-be-used-to-optimize-some-gas-usage) | 7 | 14700 |
|[G-3] Modulus operations that could be unchecked | [Modulus operations that could be unchecked](#modulus-operations-that-could-be-unchecked) | 2 | 170 |
|[G-4] Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 1 | - |
|[G-5] Avoid Unnecessary Computation Inside Loops | [Avoid Unnecessary Computation Inside Loops](#avoid-unnecessary-computation-inside-loops) | 3 | - |

Total: 5 issues


## Modulus operations that could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 170

### Description
Modulus operations should be unchecked to save gas since they cannot overflow or underflow. Execution of modulus operations outside `unchecked` blocks adds nothing but overhead. Saves about 30 gas.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: smart-contracts/XRandoms.sol

41    uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100
```
 should be unchecked

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L41](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L41)


#

```
File: smart-contracts/XRandoms.sol

36    uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000
```
 should be unchecked

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L36](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L36)


</details>

# 



## Cache External Calls in Loops
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 300

### Description
This detector identifies instances of repeated external calls within loops. By moving the first external call outside of the loop and using low-level calls inside the loop, it is possible to save gas by avoiding repeated EXTCODESIZE opcodes, as there is no need to check again if the contract exists. 

Note: This detector only triggers if the function call does not return any value.

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: smart-contracts/AuctionDemo.sol

112    IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid)
```

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112)


#

```
File: smart-contracts/MinterContract.sol

236    gencore.mint(mintIndex, mintingAddress, _mintTo, tokData, _saltfun_o, col, phase)
```

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L236](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L236)


#

```
File: smart-contracts/MinterContract.sol

189    gencore.airDropTokens(mintIndex, _recipients[y], _tokenData[y], _saltfun_o[y], _collectionID)
```

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L189](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L189)


</details>

# 


## Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 14700

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 7 instances of this issue:

</summary>

###
#

```
File: smart-contracts/AuctionDemo.sol

105    require(block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false && minter.getAuctionStatus(_tokenid) == true)
```


```
 // @audit: Switch minter.getAuctionStatus(_tokenid) == true && block.timestamp >= minter.getAuctionEndTime(_tokenid) && auctionClaim[_tokenid] == false 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L105)


#

```
File: smart-contracts/AuctionDemo.sol

111    auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true
```


```
 // @audit: Switch auctionInfoData[_tokenid][i].status == true && auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L111](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L111)


#

```
File: smart-contracts/NextGenAdmins.sol

32    require((adminPermissions[msg.sender] == true) || (_msgSender()== owner()), "Not allowed")
```


```
 // @audit: Switch (_msgSender() == owner()) || (adminPermissions[msg.sender] == true) 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L32)


#

```
File: smart-contracts/NextGenCore.sol

348    onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000
```


```
 // @audit: Switch tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000 && onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L348](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L348)


#

```
File: smart-contracts/NextGenCore.sol

345    onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000
```


```
 // @audit: Switch tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000 && onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L345](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L345)


#

```
File: smart-contracts/NextGenCore.sol

148    require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed")
```


```
 // @audit: Switch (_collectionTotalSupply <= 10000000000) && (isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L148](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L148)


#

```
File: smart-contracts/MinterContract.sol

540    collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime
```


```
 // @audit: Switch block.timestamp < collectionPhases[_collectionId].publicEndTime && collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime 
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L540](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L540)


</details>

# 


## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: smart-contracts/MinterContract.sol

329    require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs")
```

[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L329](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L329)


</details>

# 


## Avoid Unnecessary Computation Inside Loops
- Severity: Gas Optimization
- Confidence: High

### Description
This detector identifies instances of computations being performed inside loops that could be performed outside the loop to save gas.

Unnecessary computation inside loops:

Any computation performed inside a loop that doesn't change with each iteration can be moved outside the loop to save gas.This detector identifies instances where the following unnecessary computations are performed inside loops:

- 1 Local variables are read inside loops but never modified within the same loop, indicating they could be cached outside the loop.

- 2 Computations involving constant expressions (literals) which result in the same output in every loop iteration.

By addressing these issues, gas usage can be optimized.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: smart-contracts/MinterContract.sol

185    collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1
```
The following computations can be cached outside of loops for gas optimization `gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID)`<br>.
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L185](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L185)


#

```
File: smart-contracts/MinterContract.sol

188    uint256 mintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID)
```
The following computations can be cached outside of loops for gas optimization `gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID)`<br>.
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L188](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L188)


#

```
File: smart-contracts/MinterContract.sol

235    uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col)
```
The following computations can be cached outside of loops for gas optimization `gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col)`<br>.
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L235](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L235)


</details>

# 

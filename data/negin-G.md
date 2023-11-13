
## Summary  
  
### Gas Optimizations  
| |Issue|Instances|  
|-|:-|:-:|  
| [G&#x2011;01] | Duplicative and redundant memory uint definition costs additive gas | 1 |  
| [G&#x2011;02] | Extra gas cost due to multiplication to number `1`  | 1 |   
 
  
  
Total: 2 instances over 2 issues with around **200 gas** saved  
  
The table above as well as its gas numbers are created by excluding the **automatic findings** which are not included.  
  
  
  
  
  
## Gas Optimizations  
  
### [G&#x2011;01] Duplicative and redundant memory uint definition costs additive gas
Defining and assigning a `uint` variable inside a function block is in such a way that a memory slot is allocated, the value that is intended to be assigned is static-called, and then it is copied into that specific memory slot. These operations cost excess gas and decreases the efficiency of a function. We can see already that the lines #362 and #359 of the function `burnOrSwapExternalToMint()` in the contract `MinterContract`:

```Solidity
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
        gencore.mint(mintIndex, mintingAddress, ownerOfToken, tokData, _saltfun_o, col, phase);
        collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
```
  The `collectionTokenMintIndex` and `mintIndex` are identically defined and assigned which increases the gas consumption.
  
*There is 1 instance of this issue:*  
  
```solidity  
        uint256 collectionTokenMintIndex;
        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
        uint256 mintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);  
```  
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L359](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L359)
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L362)    
  
  
  
### [G&#x2011;02] Extra gas cost due to multiplication to number `1`
Multiplication of a number into `1` equals to that number. In fact, the number `1` is a multiplication-neutral number. This means that the multiplication of a number into `1` is a redundant work which costs extra gas cost. As we can see in the function `burnOrSwapExternalToMint()`, we have such an operation inside it:

```Solidity
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```
This multiplication consumes excess gas, and removing the multiplication operation will enhance the efficiency of the function.
  
*There is 1 instance of this issue:*  
  
```Solidity
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```
[https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361)  
  
  
 
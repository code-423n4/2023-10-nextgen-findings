### **The contract didn’t follow the Solidity order of layout**

In auctionDemo.sol, the contract didn’t follow the order of layout, Type declarations, State variables, Events, Errors, Modifiers, and Functions. **[Documentation](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout)** Events should be coming before state variables.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L18

In NextGenRandomizerRNG.sol, there is also a poor arrangement of layout. It doesn’t follow the solidity guide.  **[Documentation](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout)**

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L18

### ******Failure to return an error value******

In auctionDemo.sol/participateToAuction(), the require check fails to return an error value.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L58

### ******************************Unbounded Array******************************

In MinterContract.sol/mint(), there needed to be a proper check on the array length of the number of tokens.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L234

### **********************************Possible Re-enterancy**********************************

In the MinterContract.sol/burnOrSwapExternalToMint(), there is no proper follow of check→effect→interaction before sending that NFT with safeTransferFrom, I will advise the contract show use the re-enteracy guide.

### ****************************************No Address check.****************************************

In MinterContract.sol/payArtist(), there was no address check on address _team1 and _team2. If a zero address is filled in by mistake that will to a serious loss of funds.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L415

In MinterContract.sol/setCollectionCosts(), there was no address check on address _delAddress.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L157

### ********Wrong mutability usage********

In MinterContract.sol/isMinterContract(), the function used view mutability to return True variable instead of using pure.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L506

In NextGenAdmins.sol/isAdminContract() the function used view mutability to return True variable instead of using pure.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L83

In RandomizerNXT.sol/isRandomizerContract() the function used view mutability to return True variable instead of using pure.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L62

In XRandoms.sol/returnIndex() 

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L45

### **Loss of precision in division**

In  MinterContract.sol/getPrice(),

Solidity doesn't support fractions, so divisions by large numbers could result in the quotient being zero.

To avoid this, it's recommended to require a minimum numerator amount to ensure that it is always greater than the denominator.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L536

### **No address Check**

The Admin contract is one of the vital contracts in the project, so it is expected that proper sanity should be done.

In NextGenAdmins.sol/registerAdmin, there is no address check if a zero address is mistakenly set it will jeopardize the hold protocol. 

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L38

In NextGenAdmins.sol/functionAdmin, there is no address check if a zero address.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L44

In NextGenAdmins.sol/registerBatchFunctionAdmin, there is no address check if a zero address.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L50

In NextGenCore.sol/setCollectionData() there is no address check if a zero address.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147

### **Unbounded Length**

In NextGenAdmins.sol/registerBatchFunctionAdmin(), there was no proper check on the array length of the selectors.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L51

### State variables that are set in the constructor should be declared as immutable.

To access state variables within a function, an SLOAD operation is typically required. However, for immutable variables, you can access them directly, eliminating the need for an SLOAD operation and consequently reducing gas costs. Since these state variables are only assigned in the constructor, it's worth considering declaring them as immutable.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L22

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L23
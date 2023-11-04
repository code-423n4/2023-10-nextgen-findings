## [G-01] DEFINE CONSTRUCTOR AS PAYABLE
**Impact**
Developers can save around 10 opcodes and some gas if the constructors are defined as payable.
However, it should be noted that it comes with risks because payable constructors can accept ETH during deployment.
**Remediation**
It is suggested to mark the constructors as payable to save some gas. 
Make sure it does not lead to any adverse effects in case an upgrade pattern is involved.
**Locations**
```txt
smart-contracts/AuctionDemo.sol#L36-L40
smart-contracts/RandomizerNXT.sol#L25-L30
smart-contracts/NextGenAdmins.sol#L26-L28
smart-contracts/RandomizerRNG.sol#L29-L33
smart-contracts/RandomizerVRF.sol#L39-L45
smart-contracts/NextGenCore.sol#L108-L112
smart-contracts/MinterContract.sol#L129-L133
```
## [G-02] CHEAPER INEQUALITIES IN REQUIRE()
**Impact**
The contract was found to be performing comparisons using inequalities inside the require statement. 
When inside the require statements, non-strict inequalities (>=, <=) are usually costlier than strict equalities (>, <).
**Remediation**
It is recommended to go through the code logic, and, if possible, modify the non-strict inequalities with the strict ones to save ~3 gas as long as the logic of the code is not affected.
**Locations**
```txt
smart-contracts/AuctionDemo.sol#L58-L58
smart-contracts/AuctionDemo.sol#L105-L105
smart-contracts/AuctionDemo.sol#L125-L125
smart-contracts/AuctionDemo.sol#L135-L135
smart-contracts/NextGenCore.sol#L148-L148
smart-contracts/NextGenCore.sol#L206-L206
smart-contracts/MinterContract.sol#L186-L186
smart-contracts/MinterContract.sol#L213-L213
smart-contracts/MinterContract.sol#L217-L217
smart-contracts/MinterContract.sol#L223-L223
smart-contracts/MinterContract.sol#L224-L224
smart-contracts/MinterContract.sol#L232-L232
smart-contracts/MinterContract.sol#L233-L233
smart-contracts/MinterContract.sol#L251-L251
smart-contracts/MinterContract.sol#L260-L260
smart-contracts/MinterContract.sol#L261-L261
smart-contracts/MinterContract.sol#L265-L265
smart-contracts/MinterContract.sol#L266-L266
smart-contracts/MinterContract.sol#L280-L280
smart-contracts/MinterContract.sol#L294-L294
smart-contracts/MinterContract.sol#L339-L339
smart-contracts/MinterContract.sol#L360-L360
smart-contracts/MinterContract.sol#L361-L361
```
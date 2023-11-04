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
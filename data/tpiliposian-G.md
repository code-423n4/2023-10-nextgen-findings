# 1. Double storage storing leads to extensive gas usage

## Description

In the contracts `RandomizerNXT.sol`, `RandomizerVRF.sol` and `RandomizerRNG.sol` there is a `_gencore` parameter in the constructor, that is stored in two different storage slots: first in `gencore`, second in `gencoreContract` thus wasting a lot of gas.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L28-L29

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L42-L43

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L30-L31

```solidity
    constructor(address _gencore, address _adminsContract, address _arRNG) ArrngConsumer(_arRNG) {
        gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
        adminsContract = INextGenAdmins(_adminsContract);
    }
```

## Remediation

By removing one of these redundant storage variables, you will save on gas costs and storage space while maintaining the necessary functionality.
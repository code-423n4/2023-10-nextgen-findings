## [G‑01] Either `address gencore` or `INextGenCore public gencoreContract` should be removed from RandomizerNXT, RandomizerRNG and RandomizerVRF contracts.

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L22-L23

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerRNG.sol#L21-L22

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerVRF.sol#L35-L36

Declaring both the address and the contract instance will use 2 storage slots instead of one. The constructor will assign 2 values with no reason : 

```
gencore = _gencore;
gencoreContract = INextGenCore(_gencore);
```

Moreover, the `updateCoreContract(address)` function updates both values, while only one would be enough.

If you only keep the `address gencore` variable, just wrap it into a contract type when you need to call a function on it. While RandomizerNXT contract doesn't even use `gencoreContract` and creates this variable with no reason, RandomizerRNG and RandomizerVRF contracts only use it once in `fulfillRandomWords()`. You could just rewrite the call to `setTokenHash()` as follows : 

```
INextGenCore(gencore).setTokenHash(...)
```

## [G‑02] 
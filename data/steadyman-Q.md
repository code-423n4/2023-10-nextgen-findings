## [Insecure random number](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L28)

Severity: Low

Vulnerability: Choosing too low requestConfirmations will result in insecure random numbers.Please refer to the [chain.link](https://docs.chain.link/vrf/v2/getting-started) document for details.


```
uint16 public requestConfirmations = 3;
```

Fix: Use a higher requestConfirmations, recommended 12.
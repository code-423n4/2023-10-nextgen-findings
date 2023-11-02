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

## [G‑02] Useless `tokenToRequest` mapping in RandomizerRNG and RandomizerVRF contracts should be removed

Both RandomizerRNG and RandomizerVRF contracts declare a mapping `mapping(uint256 => uint256) public tokenToRequest`, and it is only use to store `requestId` in the `requestRandomWords()` function. But this value is never used, and I don't see any utility in storing the requestId corresponding to a minted token in the Randomizer contract.

While the bot report suggests to merge these following 2 mappings using struct/nested mappings in both contracts (N33 and G04 issues) : 

```
    mapping(uint256 => uint256) public tokenToRequest;
    mapping(uint256 => uint256) public tokenIdToCollection;
```

I propose to simply remove `tokenToRequest` variable, as it is unused and has no utility. This will save gas compared to creating a merged data structure without the need to do it.

## [G‑03]





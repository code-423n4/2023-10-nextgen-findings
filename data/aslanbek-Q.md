# [L-01] Mints in the first second of the public-only sale will revert

MinterContract#mint uses non-strict inequality, and getPrice uses strict one. 

Because there's no mention of the restrictoins for allowlistStartTime and allowlistEndTime, and because the test file uses allowlistStartTime = allowlistEndTime = publicStartTime, it is reasonable to assume that it would have occured over the lifetime of the protocol. 

# [L-02] Dead code in RandomizerVRF#fulfillRandomWords

[RandomizerVRF.sol#L66](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L66)

```
bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))
```

Here's what this part of the code does:

1. abi.encodePacked returns a `bytes memory` value, first 32 bytes of which will be `_randomWords[0]` (ChainlinkVRF was previously requested 1 random number)
2. the `bytes memory` value is then cast to bytes32. Casting to bytes32 takes the first 32 bytes and truncates the rest.

Thus, the code 

```
bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId]))
```

is essentially the same as 
```
bytes32(_randomWords[0])
```
It may seem that the intent was to use keccak256, but the Sponsor stated that they wanted to use only randomWords[0]. Therefore, the original code contains only a security-insensitive redundancy.
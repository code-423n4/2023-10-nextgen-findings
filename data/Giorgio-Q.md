## Wrong keyhash in `NextGenRandomizerVRF`

## Description
In the `NextGenRandomizerVRF.sol` contract there is a keyhash that is defined as a state variable. This keyhash doesn't match mainnet keyhashes. This might lead to unexpected reverts.

## Links

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L26C1-L27

## Fix

Consider initializing the contract with mainnet keyhashes. Here is a list of valid available keyhashes.

```
200 gwei Key Hash	
0x8af398995b04c28e9951adb9721ef74c74f93e6a478f39e7e0777be13527e7ef

500 gwei Key Hash	
0xff8dedfbfa60af186cf3c830acbc32c05aae823045ae5ea7da1e45fbfaba4f92

1000 gwei Key Hash	
0x9fe0eebf5e446e3c998ec9bb19951541aee00bb90ea201ae456421a2ded86805
```
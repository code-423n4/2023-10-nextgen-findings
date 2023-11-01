## Summary
[I-1] Typo.
[I-2] Leave empty lines for better code readability.
[I-3] Unnecessary parentheses.
[L-1] Wrong `keyHash` address.

### [I-1] Typo.
```solidity
@audit should be auctionInfoStruct
struct auctionInfoStru
```

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/AuctionDemo.sol#L43

```solidity
@audit should be tokenId
function requestRandomWords(uint256 tokenid) public {
```

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/RandomizerVRF.sol#L52

### [I-2] Leave empty lines for better code readability.
It is better to leave some empty spaces in the functions for better code readability. In the `MinterContract` it is very hard to read the code, because there are not empty lines.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol

### [I-3] Unnecessary parentheses.
Remove the parentheses for better code readability.

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/NextGenCore.sol#L267

### [L-1] Wrong `keyHash` address.
The `keyHash` address that is used is the `Goerli` one. There is a `updatecallbackGasLimitAndkeyHash` that allows the admin to change the `keyHash` and the `callbackGasLimit`, but still it is an issue.

```solidity
bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;
```

https://github.com/code-423n4/2023-10-nextgen/blob/ff8cfc5529ee4a567e1ce1533b4651d6626d1def/smart-contracts/RandomizerVRF.sol#L26
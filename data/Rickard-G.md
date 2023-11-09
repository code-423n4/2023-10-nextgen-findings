## [G-01] Use indexed events for value types as they are less costly compared to non-indexed ones
## Relevant GitHub Links
[https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerRNG.sol#L24](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerRNG.sol#L24)

[https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerVRF.sol#L20](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerVRF.sol#L20)

[https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L124-L126](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L124-L126)
## Summary
## Vulnerability Details
Using the `indexed` keyword for [value types](https://docs.soliditylang.org/en/v0.8.21/types.html#value-types) (`bool/int/address/string/bytes`) saves gas costs, as seen in [this example](https://gist.github.com/0xxfu/c292a65ecb61cae6fd2090366ea0877e).

However, this is only the case for value types, whereas indexing [reference types](https://docs.soliditylang.org/en/v0.8.21/types.html#reference-types) (`array/struct`) are more expensive than their unindexed version.

There are `6` instances of this issue:

- The following variables should be indexed in [NextGenRandomizerRNG.Withdraw(address, bool, uint256)](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerRNG.sol#L24):
    - status
- The following variables should be indexed in [NextGenRandomizerVRF.RequestFulfilled(uint256, uint256[])](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/RandomizerVRF.sol#L20):
    - requestId
    - randomWords
- The following variables should be indexed in [MinterContract.PayArtist(address, bool, uint256)](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L124):
    - status
- The following variables should be indexed in [MinterContract.PayTeam(address, bool, uint256)](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L125):
    - status
- The following variables should be indexed in [MinterContract.Withdraw(address, bool, uint256)](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L126):
    - status
## Impact
## Tools Used
## Recommendations
Using the `indexed` keyword for values types `bool/int/address/string/bytes` in event.
## [G-02] Modifier gas optimization for onlyOwner modifier
## Relevant GitHub Links
[https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenAdmins.sol#L38](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenAdmins.sol#L38)

[https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/Ownable.sol#L36-L53](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/Ownable.sol#L36-L53)

[https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d6b63a48ba440ad8d551383697db6e5b0ef84137/contracts/access/Ownable.sol#L45-L64](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d6b63a48ba440ad8d551383697db6e5b0ef84137/contracts/access/Ownable.sol#L45-L64)
## Summary
The require statement should be replaced with if check + custom error. This reduces the size of compiled contracts that use the modifiers. The best way of implementing this is presented in OZ's Ownable.sol ([Link 3]((https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d6b63a48ba440ad8d551383697db6e5b0ef84137/contracts/access/Ownable.sol#L45-L64))):
````diff
    modifier onlyOwner() {
        _checkOwner();
        _;
    }

    function _checkOwner() internal view virtual {
-        require(owner() == _msgSender(), "Ownable: caller is not the owner");
+        if (owner() != _msgSender()) {
+           revert OwnableUnauthorizedAccount(_msgSender());
    }
````

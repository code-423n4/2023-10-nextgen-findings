# Low/QA Issues

## [L-1] the `keyhash` used to requestRandomWords is set to Goerli Network instead of Ethereum Mainnet

### Vulnerability Details
The public `keyHash` variable is set to specific value (0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15), which is intended for use on the Goerli network. However, the contract is supposed to be deployed on the Mainnet, and the keyHash value is incompatible with the Mainnet's Chainlink VRF (Verifiable Random Function) service. Consequently, any attempts to use the requestRandomWords function for VRF v2 on the Mainnet will fail due to this mismatch.

### Affected Lines of Code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L26

### Recommendation
To resolve this issue and ensure the smart contract's compatibility with the Mainnet, it is essential to update the `keyHash` variable to the appropriate value provided by Chainlink for the Mainnet VRF service. Additionally, the updateCallbackGasLimitAndKeyHash function can be utilized to allow for dynamic updates of this value in the future, should it need to be changed again.

---
## [L-2] `addRandomizer` function accepts any arbitray address as the randomizer contract

### Vulnerability Details
The `addRandomizer` function currently allows an administrator to associate any arbitrary address with a collection as long as the associated contract returns true for the `isRandomizerContract()` function. This approach lacks proper validation and control over which randomizer contract is associated with the collection, potentially leading to incorrect configurations.

### Affected Lines of Code
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L168-L174

### Recommendation
To address these issues and enhance security, it is recommended to introduce an enum to represent the available randomizer options (e.g., RandomizerRNG, RandomizerVRF, RandomizerNXT). The admin select the proper enum value to assign the randomizer contract for a collection.

---

## [L-3] `randomPool#getWord` function return the same value for id == 0 and 1

### Vulnerability Details
```solidity
File: XRandoms.sol
28:         if (id==0) {
29:             return wordsList[id];
30:         } else {
31:             return wordsList[id - 1];
32:         }
```
This would result in the appearance word "Acai" two times more frequently than other words, while the word "Watermelon" would never appear as a return value.

### Recommendation

Consider removing the `else` branch in `randomPool#getWord`.

---

## [L-4] `fulfillRandomWords` function in `RandomizerRNG` and `RandomizerVRF` contracts could set the same hash for two different tokens

### Affected Lines of Code
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L49
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L66
```solidity
File: RandomizerRNG.sol
48:     function fulfillRandomWords(uint256 id, uint256[] memory numbers) internal override {
49:         gencoreContract.setTokenHash(tokenIdToCollection[requestToToken[id]], requestToToken[id], bytes32(abi.encodePacked(numbers,requestToToken[id])));
50:     }
```

### Vulnerability Details
`bytes32(abi.encodePacked(numbers,requestToToken[id]))` this code expects to return a mix of returned random numbers and tokens ID, it's probably used to guarantee that two different tokens would never be filled with the same hash. However, casting to `bytes32` would cut off any data that goes after the first random number. This would result in a hash collision when two different tokens have the same hash in case if random source would ever return the same random number. 

### Recommendation 

Consider setting the `keccak256` hash of tokenId and random number mix instead of its `bytes32` casted value.
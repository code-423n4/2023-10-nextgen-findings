## ./XRandom.sol
  - state mutability can be restricted to pure for function `returnIndex(uint256 id) public view returns (string memory)`


## ./RandomizerNXT.sol
  - state mutability can be restricted to pure for function `isRandomizerContract() external view returns (bool)`
  - Zero address check missing in following functions
    - `updateRandomsContract(address _randoms)` 
    - `updateAdminsContract(address _admin)`
    - `updateCoreContract(address _gencore)`
  - Unused function parameter `(uint256 _saltfun_o)` in `calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o)`

## ./RandomizerVRF.sol
  - state mutability can be restricted to pure for function `isRandomizerContract() external view returns (bool)`
  - Zero address check missing in following functios
    - `updateAdminContract(address _newadminsContract)`
    - `updateCoreContract(address _gencore)`
  - in function `fulfillRandomWords(uint256 _requestId, uint256[] memory _randomWords)` the use of `bytes32(abi.encodePacked(_randomWords,requestToToken[_requestId])` is unnecessary, as VRF will return uin256 numbers (equal to 32 bytes), thus effect of `requestToToken[_requestId]` is never reflected in bytes32


## ./RandomizerRNG.sol
  - state mutability can be restricted to pure for function `isRandomizerContract() external view returns (bool)`
  - Zero address check missing in following functios
    - `updateAdminContract(address _newadminsContract)`
    - `updateCoreContract(address _gencore)`
  - Unused function parameter `(uint256 _saltfun_o)` in `calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o)`


## ./MinterContract.sol
  - unnecessary multiplication by 1 `require(msg.value >= (getPrice(col) * 1), "Wrong ETH");` 
  - function `payArtist` does not handle the dust 
  - line 119: variable `IDelegationManagementContract private dmc` should be immutable;

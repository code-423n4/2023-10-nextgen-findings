**NextGen QA**

L1 - `setCollectionCosts` should validate that _collectionID is a valid value

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157
   
   - **After:**
     ```solidity
         require(_collectionID > 0, "Invalid collection ID"); 
     ```
     
l2- `_delAddress` should not be a zero address.
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157

    ```solidity
         require(_delAddress != address(0), "Invalid delegate address");
    ```

l3- `_collectionID` should be validated in `setCollectionPhases` function
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L170

- **After:**
     ```solidity
         require(_collectionID > 0, "Invalid collection ID"); 
     ```
l4 - Add checks for valid time ranges
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L170

    ```solidity
    require(_allowlistStartTime < _allowlistEndTime, "Invalid allowlist time range");
    require(_publicStartTime < _publicEndTime, "Invalid public time range");
    ```
     
l5- `airDropTokens` should validate that the lengths of `_recipients`, `_tokenData`, `_saltfun_o`, and `_numberOfTokens` arrays are equal.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L181
  
   - **After:**
     ```solidity
         require(_recipients.length == _tokenData.length && _recipients.length == _saltfun_o.length && _recipients.length == _numberOfTokens.length, "Array lengths mismatch");
         ...
     }
     ```
l6- `mint` should validate `_collectionID`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196

```solidity
require(_collectionID > 0, "Invalid collection ID");
```
l7- check `_mintTo` for zero address

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196

```solidity
 require(_mintTo != address(0), "Zero minting address");
```
l8- Ensure `_numberOfTokens` and `_maxAllowance` are valid

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196

```solidity
 require(_numberOfTokens > 0, "Number of tokens must be positive");
 require(_maxAllowance > 0, "Number of tokens must be positive");
```
l9- Ensure numerical values like `_tokenId`, `_burnCollectionID`, `_mintCollectionID`, and `_auctionEndTime` are valid in `burnToMint` function.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L258

```solidity
     function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
         require(_burnCollectionID > 0 && _mintCollectionID > 0, "Invalid collection ID");
         require(_tokenId > 0, "Invalid token ID");
         ...
     }
     ```

l10- Validate `_recipient` is not a zero address in `mintAndAuction` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276

```solidity
require(_recipient != address(0), "Invalid recipient address");
```

l11- Ensure `_collectionID` is valid in `mintAndAuction` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276

```solidity
 require(_collectionID > 0, "Invalid collection ID");
```

l12- Ensure that `_auctionEndTime` is in the future in `mintAndAuction` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L276

```solidity
 require(_auctionEndTime > block.timestamp, "Invalid auction end time");
```
l13- Validate `_erc721Collection` is not a zero address in `burnOrSwapExternalToMint` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326

```solidity
 require(_erc721Collection != address(0), "Invalid ERC721 collection address");
```
l14- Ensure collection Ids are valid in `burnOrSwapExternalToMint` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326

```solidity
 require(_burnCollectionID > 0 && _mintCollectionID > 0, "Invalid collection IDs");
```
l15- Ensure _tokenData is not empty in `burnOrSwapExternalToMint` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L326

```solidity
 require(bytes(_tokenData).length > 0, "Token data cannot be empty");
```

l16- Add zero address check for _team1 and _team2

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L415

    ```solidity
    require(_team1 != address(0), "Invalid team1 address"); // Zero address check for _team1
    require(_team2 != address(0), "Invalid team2 address"); // Zero address check for _team2
    ```
l17- Validate addresses are not zero addresses in `proposeSecondaryAddressesAndPercentages`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L394C14-L394C53

```solidity
    require(_secondaryAdd1 != address(0), "Invalid _secondaryAdd1 address"); // Zero address check for _secondaryAdd1
    require(_secondaryAdd1 != address(0), "Invalid _secondaryAdd2 address"); // Zero address check for _secondaryAdd2
    require(_secondaryAdd1 != address(0), "Invalid _secondaryAdd3 address"); // Zero address check for _secondaryAdd3
    ```
    
     
L18- `payArtist` function makes external calls. Ensure the success of these calls is checked.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L415
   
   - **After:**
     ```solidity
     (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
     require(success1, "Payment to primaryAdd1 failed");
     ```


L19- `updateCoreContract` should validate that the input address is not the zero address.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L448
  
   - **After:**
     ```solidity
         require(_gencore != address(0), "Zero address provided");
     ```
     
L20- `updateAdminContract` should validate that the input address is not the zero address.  

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L454
   
     ```solidity
         require(_newadminsContract != address(0), "Zero address provided");
     ```

L21 `emergencyWithdraw` should check for the success of the Ethereum transfer.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L461
   
   - **After:**
     ```solidity
     (bool success, ) = payable(admin).call{value: balance}("");
     require(success, "Transfer failed");
     ```
L22- `setCollectionData`: Validate `_collectionArtistAddress` is not a zero address

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147

```solidity
require(_collectionArtistAddress != address(0), "Zero artist address");
```

L23- `setCollectionData`: Validate `_maxCollectionPurchases`, `_collectionTotalSupply`, `_setFinalSupplyTimeAfterMint` are within valid ranges.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147

```solidity
require(_maxCollectionPurchases > 0, "Invalid max purchases");
```

L24- `addRandomizer`: Validate `_randomizerContract` is not a zero address.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L170

```solidity
require(_randomizerContract != address(0), "Zero address");
```

L25- `airDropTokens` function should check for zero address

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178

```solidity
require(_receipient != address(0), "Zero address");
```
L26- `mint` function should check for zero address

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L189

```solidity
require(_mintingAddress != address(0), "Zero address");
require(_mintTo != address(0), "Zero address");
```
L27- `_mintProcessing` function should check for zero address
```solidity
require(_receipient != address(0), "Zero address");
```

L28- `artistSignature`: Ensure `_signature` is not empty.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L257
```solidity

```

L29- `changeTokenData`: Check for string emptiness.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L273

```solidity
 require(bytes(newData).length > 0, "newData cannot be empty");  // Check for string emptiness
```

L30- Ensure `_newCollectionScript` matches the expected size based on the `_index` in `updateCollectionInfo` function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L238

```solidity
require((_index == 1000 && _newCollectionScript.length > 0) || (_index != 1000 && _newCollectionScript.length == 1), "Invalid script size");
```
L31- `updateImagesAndAttributes`: Validate `_tokenId`, `_images`, and `_attributes` for correct lengths and ensure none of the token IDs are zero.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L281

```solidity
     function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(...) {
         require(_tokenId.length == _images.length && _tokenId.length == _attributes.length, "Array lengths mismatch");
         for (uint256 x = 0; x < _tokenId.length; x++) {
             require(_tokenId[x] > 0, "Invalid token ID");
             ...
         }
     }
```

L32- `addMinterContract` should validate the input address is not the zero address

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L315

```solidity
 require(_minterContract != address(0), "Zero address provided");
```

L33- `updateAdminContract` should validate the input address is not the zero address

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L322

```solidity
require(_newadminsContract != address(0), "Zero address provided");
```


L34- `setDefaultRoyalties`: Validate `_royaltyAddress` is not a zero address 

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L329

     ```solidity
         require(_royaltyAddress != address(0), "Invalid royalty address");
     ```



L35- `setDefaultRoyalties`: Validate `_bps` (basis points) is within a valid range.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L329

     ```solidity
         require(_bps <= 10000, "Basis points exceed limit"); // Assuming 10000 bps is the maximum (100%)
     }
     ```

L36- `registerAdmin`, `registerFunctionAdmin`, `registerBatchFunctionAdmin`, and `registerCollectionAdmin` do not validate the input address to ensure it is not the zero address. 

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L38
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L44
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L50
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L58
   
     ```solidity
     function registerAdmin(address _admin, bool _status) public onlyOwner {
         require(_admin != address(0), "Zero address provided");
         ...
     }
     ```

L37- The `registerBatchFunctionAdmin` function should validate that the `_selector` array is not empty.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L50

     ```solidity
     function registerBatchFunctionAdmin(address _address, bytes4[] memory _selector, bool _status) public AdminRequired {
         require(_selector.length > 0, "Selector array is empty");
         ...
     }
     ```
L38- The `updateRandomsContract`, `updateAdminsContract`, and `updateCoreContract` functions should validate the input address to ensure it is not the zero address.
 
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L41
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L45
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L49
   
     ```solidity
     function updateRandomsContract(address _randoms) public FunctionAdminRequired(this.updateRandomsContract.selector) {
         require(_randoms != address(0), "Zero address provided");
         ...
     }
     ```
L39- `requestRandomWords` does not validate the `tokenid`.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L52
   
     ```solidity
     function requestRandomWords(uint256 tokenid) public {
         require(tokenid > 0, "Invalid token ID"); // Added input validation
         ...
     }
     ```

L40- Functions `updateAdminContract` and `updateCoreContract` should validate the input address is not the zero address.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L94
    
    - **After:**
      ```solidity
      function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) {
          require(_gencore != address(0), "Zero address provided");
          ...
      }
      ```
L41- `requestRandomWords` should validate that `_ethRequired` is a sensible amount.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L40

   - **After:**
     ```solidity
     function requestRandomWords(uint256 tokenid, uint256 _ethRequired) public payable {
         require(_ethRequired > 0, "ETH required is zero");
         ...
     }
     ```

L42- Functions like `updateAdminContract`, `updateCoreContract`, and `updateRNGCost` could potentially require an `onlyOwner` modifier for enhanced security, depending on the contract's design.


L43- The `emergencyWithdraw` function should have a check to ensure that the call to transfer Ether is successful.
   
   - **After:**
     ```solidity
     function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
         ...
         (bool success, ) = payable(admin).call{value: balance}("");
         require(success, "Transfer failed");
         ...
     }
     ```

L44. Functions `updateAdminContract` and `updateCoreContract` should validate the input address is not the zero address.
    
    - **After:**
      ```solidity
      function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) {
          require(_gencore != address(0), "Zero address provided");
          ...
      }
      ```

L44. `returnIndex` function lacks input validation for `id`. It should verify that `id` is within the bounds of the `wordsList` array.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L45
   
   - **After:**
     ```solidity
     function returnIndex(uint256 id) public view returns (string memory) {
         require(id < 100, "Invalid ID"); // Added input validation
         return getWord(id);
     }
     ```

     

L45. The `getWord` function could benefit from a bounds check for the `id` parameter to ensure it doesn't exceed the array length. However, the current logic already handles the zero index case.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L15

     ```solidity
     function getWord(uint256 id) private pure returns (string memory) {
         require(id < 100, "ID out of bounds"); // Added bounds check
         // ...
         if (id == 0) {
             return wordsList[id];
         } else {
             return wordsList[id - 1];
         }
     }
     ```
L46. `participateToAuction` should be checked to ensure it's a valid token ID.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L57

   - **After:**
     ```solidity
     function participateToAuction(uint256 _tokenid) public payable {
         require(_tokenid > 0, "Invalid token ID"); // Added input validation
         ...
     }
     ```

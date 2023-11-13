## [L-01] NextGenCore::reservedMaxTokensIndex to become less than already reserved indexes

Tokens are indexed sequentially starting from 0 for each collection. The next available index is tracked by the contract state variable reservedMaxTokensIndex.

However, this variable is computed as a function of the mutable collectionTotalSupply state. If collectionTotalSupply is later decreased via a call to setCollectionData, it can cause reservedMaxTokensIndex to also decrease below indexes that were already reserved for minted tokens.

This means those lower indexes could become available again and potentially be reused, creating duplicate token indexes.

The issue arises because indexing logic relies on a mutable state value. To prevent inconsistencies if supply changes later, reservedMaxTokensIndex should either be calculated independently without reference to supply, or made immutable once minting begins for a collection.

```solidity
File:  main/smart-contracts/NextGenCore.sol

    function viewTokensIndexMax(uint256 _collectionID) external view returns (uint256) {
        return(collectionAdditionalData[_collectionID].reservedMaxTokensIndex);
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L389-L391

Otherwise, future changes to supply could inadvertently introduce indexing collisions and duplicates.

## [L-02] There is no check to ensure the recipient address is valid, which could allow tokens to be sent to invalid/burn addresses.

Tokens can be minted to invalid/burn addresses as the MinterContract.sol::mint and MinterContract.sol::burnToMintfunction do not validate the `_mintTo` address parameter. This could allow tokens to be permanently burned by sending them to address(0) or other invalid recipients.

Proper input validation is required to prevent such edge cases and ensure tokens can only be minted to valid wallet addresses that can receive and transfer ownership of the tokens later on.

```solidity
File:
196    function mint(uint256 _collectionID, uint256 _numberOfTokens, uint256 _maxAllowance, string memory _tokenData, address _mintTo, bytes32[] calldata merkleProof, address _delegator, uint256 _saltfun_o) public payable {


258    function burnToMint(uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, uint256 _saltfun_o) public payable {
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L196

## [L-03] If the value of the minterContract is empty before it's initialized

The NextGenCore::mint, NextGenCore::airDropTokens, NextGenCore::burnToMint functions require the caller to be the minter contract address. However, this address is not initialized during contract construction and relies on an external addMinterContract function to set the value later on.

If mint or airDropTokens are called before addMinterContract, then the minterContract value will be empty and the caller check will not validate the sender properly. This could allow unintended actors to call the arbitrary address and mint or airdrop tokens during the period before the minter is set.

Proper initialization of dependent values is required to prevent vulnerabilities arising from missing pre-conditions and uninitialized states.

```solidity
File: main/smart-contracts/NextGenCore.sol

214        require(msg.sender == minterContract, "Caller is not the Minter Contract");

179        require(msg.sender == minterContract, "Caller is not the Minter Contract");

190        require(msg.sender == minterContract, "Caller is not the Minter Contract");


315    function addMinterContract(address _minterContract) public FunctionAdminRequired(this.addMinterContract.selector) {
316        require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter");
317        minterContract = _minterContract;
318    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L315-L318

## [L-04] Gas Limit Exceed During AirDrops

The airDropTokens function does not properly account for gas costs when distributing tokens, which could cause individual airdrop transactions to fail or the entire process to run out of gas if numreous small amounts are distributed.

Each airdrop requires gas for the function execution as well as the `_mintProcessing` and storage updates. No limits are placed on the number of airdrops or validation of sufficient gas remaining.

This presents a gas exhaustion vulnerability where the contract may be left in an undesired state if distribution fails partway through due to running out of gas. Proper gas accounting and limiting is necessary to prevent such issues.

```solidity
File: main/smart-contracts/NextGenCore.sol

    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
        if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178-L185

## [L-05] Testnet tokens can be accidentally minted on mainnet

The NextGenCore contract does not properly separate testnet and mainnet functionality. When minting or burning tokens, it does not check which network is being used. This allows tokens that were minted or burned on the testnet to also be minted or burned on mainnet (and vice versa).

This could lead to accidental inflation issues if testnet transactions were replayed on mainnet. It may be possible for an attacker to mint additional tokens on mainnet by replaying testnet transactions.

The contract needs to be updated to distinguish between testnet and mainnet when handling minting, burning, and other transactions that affect supply. A simple way would be to add a network ID check on these functions to reject non-native network calls.

## [L-06] Reliance on Randomizer Output Without Validation

The NextGenCore.sol fully relies on the output of the external randomizer contract to calculateTokenHash during the minting process, without performing any validation of the result. This could allow issues or manipulation of the randomizer to impact minting without the contract detecting it. The contract should validate the hash returned by the randomizer (VRF,NXT,RNG) by calculating the expected hash itself and comparing, to ensure the integrity of the randomization process and catch any potential issues.

```solidity
File: main/smart-contracts/NextGenCore.sol

229        collectionAdditionalData[_collectionID].randomizer.calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L229

## [L-07] Potential Array Index Out of Bounds Access in NextGenCore::updateImagesAndAttributes

The updateImagesAndAttributes function iterates through the `_tokenId` array to update metadata, but does not validate that the `_images` and `_attributes` arrays are of the same length.

This could result in an "index out of bounds" error if they are not the same size. Problems include `_images` and `_attributes` may not be the same length, The loop would access invalid indexes and fail Or silently write to the wrong metadata

To prevent this, the lengths should be checked before the loop:

```solidity
require(_tokenId.length == _images.length);
require(_tokenId.length == _attributes.length);

for (uint256 i; i < _tokenId.length; i++) {
  // access arrays
}
```

Alternatively access by index instead of length in the loop:

```solidity
for (uint256 i = 0; i < _tokenId.length; i++) {

  require(_images.length > i);
  require(_attributes.length > i);

  // access arrays
}
```

Without validating lengths first, there is potential for memory access errors or metadata corruption.

```solidity
File: smart-contracts/NextGenCore.sol

281    function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
282        for (uint256 x; x < _tokenId.length; x++) {
283            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
284            _requireMinted(_tokenId[x]);
285            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
286            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
287        }
288    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L281-L288

## [L-08] No checks to prevent same token ID being updated multiple times accidentally.

The updateImagesAndAttributes function in the NextGenCore contract allows metadata for the same token ID to be updated multiple times without checks. It does not prevent accidentally passing the same token ID into the function repeatedly. This could corrupt the metadata if one update overwrote a previous update for that token ID. The function should add checks to prevent updating metadata for a token ID that has already been passed in previously.

```solidity
File: smart-contracts/NextGenCore.sol

281    function updateImagesAndAttributes(uint256[] memory _tokenId, string[] memory _images, string[] memory _attributes) public FunctionAdminRequired(this.updateImagesAndAttributes.selector) {
282        for (uint256 x; x < _tokenId.length; x++) {
283            require(collectionFreeze[tokenIdsToCollectionIds[_tokenId[x]]] == false, "Data frozen");
284            _requireMinted(_tokenId[x]);
285            tokenImageAndAttributes[_tokenId[x]][0] = _images[x];
286            tokenImageAndAttributes[_tokenId[x]][1] = _attributes[x];
287        }
288    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L281-L288

## [L-09] The implementation of the function is missing one check for the freezeCollection()

The documentation for the freezeCollection function contains an stating it should check that the collection exists and is not already frozen. However, the implementation of the function is missing one check for the frozen.

- Documentation:

  ```
  Notes:
  This function can be called by a global admin or a function admin.
  The collection should exist and not be frozen.
  Once executed the collection's freeze status becomes true and cannot be altered.
  ```

https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/core

- Code:

  ```solidity
  File: smart-contracts/NextGenCore.sol

      function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
          require(isCollectionCreated[_collectionID] == true, "No Col");
          collectionFreeze[_collectionID] = true;
      }
  ```

  https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L292-L295

It requires the collection exists by checking isCollectionCreated, but does not check if it is already frozen before setting collectionFreeze to true. This could result in a collection being frozen multiple times accidentally.

## [L-10] Duplicate Finalization Vulnerability in setFinalSupply function

The setFinalSupply function in the NextGenCore does not validate that the collection's supply has not already been finalized before setting it. This allows an attacker or accidental duplicate calls to finalise the same collection's supply to different values, breaking the immutability of the finalized supply.

```solidity
File: main/smart-contracts/NextGenCore.sol

307    function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
308        require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
309        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
310        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
311    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L307-L311

The function should validate that a collection has not already been finalized by adding a per-collection flag or requiring a monotonically increasing nonce. Without these checks, an attacker could call the function multiple times to inconsistently modify the state. Proper validation before state changes is needed to ensure final supply values cannot be modified once set and maintain integrity of collection parameters finalized by the contract.

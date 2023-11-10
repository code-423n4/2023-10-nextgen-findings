# Report

## Low Issues

### [L-1] Floating pragma in all in-scope contracts

Pragma should be locked at `0.8.19` rather than having it floating as it is now. Locking the pragma helps ensure the contracts won't get accidentally deployed using a different version.

### [L-2] Set keyHash and callbackGasLimit in constructor in RandomizerVRF.sol

Currently, we have the ` keyHash` and `callbackGasLimit` hard-coded as follows:

```solidity
bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;
uint32 public callbackGasLimit = 40000;
```

They don't get set in the constructor. Instead, the admin has to call the function `updatecallbackGasLimitAndkeyHash` after each deployment:

```solidity
function updatecallbackGasLimitAndkeyHash(uint32 _callbackGasLimit, bytes32 _keyHash) public FunctionAdminRequired(this.updatecallbackGasLimitAndkeyHash.selector){
  callbackGasLimit = _callbackGasLimit;
  keyHash = _keyHash;
}
```

This is an unnecessary step, they could be set on deployment in the constructor, especially since the gas lane key hash is chain dependent. Also, it doesn't make sense to have `keyHash` set to Goerli's gas lane as it is now. Keep it unset by default.

### [L-3] Lack of 0 check in MinterContract:mint() and MinterContract:mintAndAuction() for collectionPhases[col].timePeriod

There is no check for 0 value on [L249](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L249), [L292](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L292), and [L546](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L546).

```solidity
uint tDiff = (block.timestamp - timeOfLastMint) / collectionPhases[col].timePeriod;
```

```solidity
tDiff = (block.timestamp - collectionPhases[_collectionId].allowlistStartTime) / collectionPhases[_collectionId].timePeriod;
```

This will cause "Division by Zero" error if executed with the `timePeriod` set to 0, consider adding a check.

## Non Critical Issues

### [NC-1] Contracts in XRandoms.sol and AuctionDemo.sol should follow pascal case convention

The recommended convention for Solidity smart contracts is to always use `PascalCase` for contract names. It is currently not followed for `randomPool` and `auctionDemo` in ` XRandoms.sol` and `AuctionDemo.sol` respectively.

### [NC-2] Use shorthand rather than checking for "== true" and "== false" everywhere

Checks against `== true` and `== false` are used across all contracts in scope. For readability and conciseness purposes, I would recommend changing all such expressions to their shorthand forms

For example, instead of using:

```solidity
isCollectionCreated[_collectionID] == true && collectionFreeze[_collectionID] == false
```

simply use:

```solidity
isCollectionCreated[_collectionID] && !collectionFreeze[_collectionID]
```

This would save a some line space.

### [NC-3] Check against address(0) instead of 0x0000000000000000000000000000000000000000000000000000000000000000

All contracts in scope check against the full address form of address 0, rather than just using `address(0)`. I would recommend changing it to `address(0)` everywhere to save space.

### [NC-4] Magic numbers in constructor when calling \_setDefaultRoyalty()

Magic numbers are passed into `_setDefaultRoyalty()` in the constructor of `NextGenCore`. I would recommend extracting those into variables with readable names to avoid confusion.

### [NC-5] Use pascal case naming convention for all structs

`PascalCase` is the recommended convention for all structs in Solidity. Currently, this convention is not followed in some places

`NextGenCore.sol:` collectionInfoStructure, collectionAdditonalDataStructure
`MinterContract.sol:` collectionPhasesDataStructure, royaltiesPrimarySplits, collectionPrimaryAddresses, royaltiesSecondarySplits, and collectionSecondaryAddresses
`AuctionDemo.sol:` auctionInfoStru

### [NC-6] Rename collectionCirculationSupply to collectionCirculatingSupply in collectionAdditonalDataStructure struct

The proper collocation is "circulating supply", not "circulation supply", so it's a good idea to rename it to `collectionCirculatingSupply`.

### [NC-7] Use camel case convention for all modifiers

All contracts in scope currently use the `PascalCase` convention for the modifiers' names. The recommended convention, though, is `camelCase`.

### [NC-8] Duplication of data in addRandomizer

In `addRandomizer`, we can see that the randomizer contract is being added twice as a in the `collectionAdditionalData` structs:

```solidity
collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract);
```

I'd recommend changing this to:

```
collectionAdditionalData[_collectionID].randomizerContract = IRandomizer(_randomizerContract);
```

As this is the convention followed in `addMinterContract()` and `updateAdminContract()`.

### [NC-9] Consider extracting all repeating event name strings into Solidity events

There are a lot of repeating event strings in `require` statements across all contracts. I would recommend extracting into native events and using those. This would prevent name inconsistencies, repetition, and potential typos.

### [NC-10] Rename retrievewereDataAdded() and wereDataAdded

`retrievewereDataAdded()` and `wereDataAdded` should be renamed to `retrieveWasDataAdded()` and `wasDataAdded` respectively.

### [NC-11] Unneeded multiplication by 1 in MinterContract:burnOrSwapExternalToMint()

There is an unneeded multiplication by one at [L361](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/MinterContract.sol#L361) in MinterContract:

```solidity
require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```

### [NC-12] AuctionDemo:participateToAuction() should be renamed to participateInAuction()

`participateToAuction()` should be renamed to `participateInAuction()`

### [NC-13] Magic address number 0x8888888888888888888888888888888888888888 in MinterContract's mint() and burnOrSwapExternalToMint()

The addresss ` 0x8888888888888888888888888888888888888888` should be extracted into a descriptive global variable and re-used in `mint()`and`burnOrSwapExternalToMint()`.

### [NC-14] Rename collectionFreeze mapping to collectionFrozen in MinterContract

Rename the mapping from `collectionFreeze` to `collectionFrozen`.

### [NC-15] Apply comments from docs to code as well

There are some really good and descriptive comments in the docs that are not present in the code. For example, this is how the `mint` function from `MinterContract` is described in the docs:

```solidity
/**
  * @dev Mint new tokens during allowlist or public minting.
  * @param _collectionID Refers to collection for which the tokens will be minted.
  * @param _numberOfTokens Refers to the number of tokens that will be minted. This number should be less than the max allowance
    during allowlist minting or less than the _maxCollectionPurchases during public minting.
  * @param _maxAllowance Refers to the max allowance per wallet during allowlist minting. For public minting this value is 0.
  * @param _tokenData Refers to the additional token data that will be stored on-chain for each minted token.
  * @param _mintTo Refers to the address that the token will be minted to.
  * @param merkleProof Refers to the set of hashes that can be used to prove a given leaf's membership in the merkle tree.
  * @param _delegator Refers to the delegator address during allowlist minting. If the minter will mint on his behalf the _delegator value
    is set to 0x0000000000000000000000000000000000000000, otherwise the delegator's address needs to be provided.
  * @param _saltfun_o Refers to an arbitrary voluntary value that can be used in the future to change or weight the minting outcomes away from purely random.
*/

function mint(
  uint256 _collectionID,
  uint256 _numberOfTokens,
  uint256 _maxAllowance,
  string _tokenData,
  address _mintTo,
  bytes32[] calldata merkleProof,
  address _delegator,
  uint256 _saltfun_o
) public payable;
```

and this is how it is in the curent code:

```solidity
// mint function

function mint(
    uint256 _collectionID,
    uint256 _numberOfTokens,
    uint256 _maxAllowance,
    string memory _tokenData,
    address _mintTo,
    bytes32[] calldata merkleProof,
    address _delegator,
    uint256 _saltfun_o
) public payable { ... }
```

# Low Priority Issues

## [L-1] Address 0 Missed Checks

### Description
The functions in the provided links do not have checks to ensure that address parameters are not the zero address.

### Affected Code
- [Function at Line 109](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L109)
- [Function at Line 330](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L330)

### Recommended Mitigation Steps
Implement address validation checks at the beginning of the functions to prevent the zero address from being used. For example:

```solidity
require(_addressParameter != address(0), "Address parameter cannot be the zero address");
```


## [L-2] Missing Non-Reentrant Modifier on Functions

### Description
The following functions are potentially vulnerable to reentrancy attacks as they invoke `mintProcessing`, which in turn calls `safeMint`. The `safeMint` function may trigger the `onERC721Received` hook on the receiver if it is a contract, allowing for reentrancy.

### Affected Code
- [Function at Line 178](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L178)
- [Function at Line 189](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L189)
- [Function at Line 213](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L213)

### Recommended Mitigation Steps
To safeguard against reentrancy, the `nonReentrant` modifier should be added to these functions. This modifier should be included in the contract from OpenZeppelin's security library or a similar implementation should be created to prevent reentrant calls.

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

// Add to contract definition
contract NextGenCore is ReentrancyGuard {
    // ...
}

// Add nonReentrant modifier to the functions
function functionName(...) public nonReentrant {
    // ...
}
```

## [L-3] Missing Collection Freeze Check in setCollectionBaseURI

### Description
The `freezeCollection` function lacks a check to ensure that the collection is not already frozen. 

### Affected Code
- [freezeCollection Function at Line 293](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L293)

### Recommended Mitigation Steps
Introduce an additional condition in the `require` statement to check the `collectionFreeze` status. The function should revert if an attempt is made to call on a frozen collection.

```solidity
    function freezeCollection(uint256 _collectionID) public FunctionAdminRequired(this.freezeCollection.selector) {
        require(isCollectionCreated[_collectionID] == true, "No Col");
        require(collectionFreeze[_collectionID] == false, "Already frozen");
        collectionFreeze[_collectionID] = true;
    }
```

## [L-4] Unnecessary Indexing of Funds in Events

### Description
The contract indexes fund values in the `AuctionCreated`, `AuctionSuccessful`, and `AuctionCancelled` events. Indexing fund values such as bid amounts or prices is generally not recommended as it does not serve a purpose for event filters and increases gas costs.

### Affected Code
- [Auction Events at Line 22-24](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L22-L24)
- [Minter Events at Line 124-126](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L124-L126)

### Recommended Mitigation Steps
Remove the `indexed` keyword from the fund-related parameters in the event definitions to reduce unnecessary gas costs and adhere to best practices. And also remove funds at all. 

# Non-critical Issues

## [N-1] Fragmented Storage of Collection Information
### Code
- [Line 41](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L41)
- [Line 65](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L65)
- [Line 71](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L71)
- [Line 83](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L83)
### Description
The contract stores collection information in a fragmented manner across multiple variables. Consolidating this information into a single structure would improve code readability and maintainability.

## [N-2] Initial Collection Index Set to 1
### Code
- [Line 110](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L110)
### Description
The initial collection index is set to 1. Starting from 0 is a more conventional approach and aligns with typical array indexing in programming.

## [N-3] Hardcoded Address in Contract
### Code
- [Line 111](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L111)
### Description
The contract hardcodes an address, which reduces flexibility. Passing the address as a parameter during contract deployment would be more adaptable to changes.

## [N-4] Unnecessary Parentheses in Code
### Code
- [Line 148](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148)
### Description
There are unnecessary parentheses in the code that do not affect the operation and could be removed for cleaner code.

## [N-5] Missing `isMinterContract` Modifier in Functions
### Code
- [Line 179](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L179)
- [Line 190](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L190)
### Description
The functions are missing the `isMinterContract` modifier which could potentially lead to unauthorized minting.

## [N-6] Sale Option Should Use Enum
### Code
- [Line 54](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L54)
### Description
The sale option is currently represented as a boolean. Using an enum would make the contract more readable and future-proof for additional sale options.

## [N-7] Function Name Misleading
### Code
- [Line 512](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L512)
### Description
The function name `getEndTime` is misleading as it pertains to the public end time. Renaming it to `getPublicEndTime` would be more descriptive and accurate.



# [G-01] Multiple storage writes of `collectionInfo` state variable

### Description
Assigning individual values to each field of struct in storage variable is expensive operation. That involves multiple write operations, which are gas expensive.

Instead of storing variables one by one, store them at once. Storing it at once will be more gas efficient

### Context

- [NextGenCore.sol#130](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L130)

### Migration steps

Create temporary memory variable to store **collectionInfoStructure** data and then store that variable into a **collectionInfo** state variable. In this way we are storing **collectionInfoStructure** data with only one *write* operation which is more gas efficient.

#### Example
```solidity
function createCollectionv2(string memory _collectionName, string memory _collectionArtist, string memory _collectionDescription, string memory _collectionWebsite, string memory _collectionLicense, string memory _collectionBaseURI, string memory _collectionLibrary, string[] memory _collectionScript) public {
      collectionInfoStructure  memory newCollection = collectionInfoStructure(
        _collectionName,
        _collectionArtist,
        _collectionDescription,
        _collectionWebsite,
        _collectionLicense,
        _collectionBaseURI,
        _collectionLibrary,
        _collectionScript
      );
      collectionInfo[newCollectionIndex] = newCollection;
      isCollectionCreated[newCollectionIndex] = true;
      newCollectionIndex = newCollectionIndex + 1;
    }
```





### Tools used

- Manual review
- Remix Ide


# [G-02] Read multiple times from state variables


### Description
There are functions which read multiple times same state variable. These functions must store data from state variable to local variable *(variable in a function)* and read from it. By reading data from local variable the function will be more gas efficient

### Context
 - [NextGenCore.sol#178](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178)
 - [NextGenCore.sol#170](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L170)
 - [NextGenCore.sol#147](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147)
 - [NextGenCore.sol#189](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L189)
 - [NextGenCore.sol#238](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L238)

### Migration steps
In a example below we are creating new local variable which will hold data inside a function.
In this way the function is more gas effecient becase we do not read every time from **collectionAdditionalData** state variable.



#### Example

```solidity
  function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {
        require(msg.sender == minterContract, "Caller is not the Minter Contract");
        collectionAdditonalDataStructure storage collectionData = collectionAdditionalData[_collectionID];

        collectionData.collectionCirculationSupply = collectionData.collectionCirculationSupply + 1;
        if (collectionData.collectionTotalSupply >= collectionData.collectionCirculationSupply) {
            _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
            tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;
        }
    }
```

### Tools used
- Manual review
- Remix Ide



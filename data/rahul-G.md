### 1. 22490 gas can be saved by removing superfluous storage variable

The `NextGenCore.sol` contract uses a storage variable called `isCollectionCreated` to check if a collection has been created. 

Alternatively, the existence of a collection can also be verified by checking if the `collectionID` is less than `newCollectionIndex`, since all existing collections will have a value that is less than `newCollectionIndex`. This optimization saves an expensive storage write during contract creation:

**STEP 1:** Remove the storage variable declaration, and write from `createCollection()`.

| **Method**       | NextGenCore:createCollection() |
|------------------|--------------------------------|
| **Before**       | 252555                         |
| **After**        | 230253 (-22302)                |
| **# Calls**      | 1                              |
| **Test passes?** | Yes                            |

```diff
__SNIP__
- mapping (uint256 => bool) private isCollectionCreated;
__SNIP__

    function createCollection(__SNIP__) {
        collectionInfo[newCollectionIndex].collectionName = _collectionName;
        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
-       isCollectionCreated[newCollectionIndex] = true;
        newCollectionIndex = newCollectionIndex + 1;
    }
__SNIP__

```

[📄 Source: smart-contracts/NextGenCore.sol#L62](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L62)
[📄 Source: smart-contracts/NextGenCore.sol#L139](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L139)

**STEP 2:** Refactor the require statements to verify using `newCollectionIndex` instead in `setCollectionData()`, `updateCollectionInfo()`, `changeMetadataView()`, and  `freezeCollection()`. It also verifies that `collectionID` greater than zero since `newCollectionIndex` starts with 1.

| **Method**       | NextGenCore:setCollectionData() |
|------------------|---------------------------------|
| **Before**       | 199078                          |
| **After**        | 199031 (-47)                    |
| **# Calls**      | 1                               |
| **Test passes?** | Yes                             |

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");

```

[📄 Source: smart-contracts/NextGenCore.sol#L148](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148)

| **Method**       | NextGenCore:updateCollectionInfo() |
|------------------|------------------------------------|
| **Before**       | 51993                              |
| **After**        | 51946 (-47)                        |
| **# Calls**      | 1                                  |
| **Test passes?** | Yes                                |

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false), "Not allowed");

```

[📄 Source: smart-contracts/NextGenCore.sol#L239](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L239)

| **Method**       | NextGenCore:changeMetadataView() |
|------------------|----------------------------------|
| **Before**       | 62755                            |
| **After**        | 62708 (-47)                      |
| **# Calls**      | 1                                |
| **Test passes?** | Yes                              |

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+ require(( _collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false), "Not allowed");

```

[📄 Source: smart-contracts/NextGenCore.sol#L267](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L267)

| **Method**       | NextGenCore:freezeCollection() |
|------------------|--------------------------------|
| **Before**       | 57073                          |
| **After**        | 57026 (-47)                    |
| **# Calls**      | 1                              |
| **Test passes?** | Yes                            |

```diff
- require(isCollectionCreated[_collectionID] == true, "No Col");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex), "No Col");
```

[📄 Source: smart-contracts/NextGenCore.sol#L293](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L293)

#### Test file for verification
```js
const {
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;

describe("verify removal of `isCollectionCreated` storage variable", async function () {
  before(async function () {
    ({ signers, contracts } = await loadFixture(fixturesDeployment));
  });

  it("SHOULD deploy contracts", async function () {
    expect(await contracts.hhAdmin.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhCore.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhDelegation.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhMinter.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandomizer.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
    expect(await contracts.hhRandoms.getAddress()).to.not.equal(
      ethers.ZeroAddress
    );
  });

  it("SHOULD revert when setting data on non existent collection", async function () {
    await expect(
      contracts.hhCore.setCollectionData(1, signers.owner.address, 50, 100, 100)
    ).to.be.revertedWith("err/freezed");
  });

  it("SHOULD revert when updating non existent collection", async function () {
    await expect(
      contracts.hhCore.updateCollectionInfo(
        1,
        "Test Collection 1",
        "Artist 1",
        "For testing",
        "www.test.com",
        "CCO",
        "https://ipfs.io/ipfs/hash/",
        "",
        999,
        ["desc"]
      )
    ).to.be.revertedWith("Not allowed");
  });

  it("SHOULD revert when changing meta data of a non existent collection", async function () {
    await expect(
      contracts.hhCore.changeMetadataView(1, true)
    ).to.be.revertedWith("Not allowed");
  });

  it("SHOULD revert when freezing non existent collection", async function () {
    await expect(contracts.hhCore.freezeCollection(1)).to.be.revertedWith(
      "No Col"
    );
  });

  it("SHOULD create a collection", async function () {
    await contracts.hhCore.createCollection(
      "Test Collection 1",
      "Artist 1",
      "For testing",
      "www.test.com",
      "CCO",
      "https://ipfs.io/ipfs/hash/",
      "",
      ["desc"]
    );
  });

  it("SHOULD succeed when setting data on an existing collection", async function () {
    await expect(
      contracts.hhCore.setCollectionData(1, signers.owner.address, 50, 100, 100)
    ).not.to.be.revertedWith("err/freezed");
  });

  it("SHOULD succeed when updating an existing collection", async function () {
    await expect(
      contracts.hhCore.updateCollectionInfo(
        1,
        "Test Collection 1",
        "Artist 1",
        "For testing",
        "www.test.com",
        "CCO",
        "https://ipfs.io/ipfs/hash/",
        "",
        999,
        ["desc"]
      )
    ).not.to.be.revertedWith("Not allowed");
  });

  it("SHOULD succeed when changing meta data of an existing collection", async function () {
    await expect(
      contracts.hhCore.changeMetadataView(1, true)
    ).not.to.be.revertedWith("Not allowed");
  });

  it("SHOULD succeed when freezing an existing collection", async function () {
    await expect(contracts.hhCore.freezeCollection(1)).not.to.be.revertedWith(
      "No Col"
    );
  });
});

```
#### Test file result
```bash
  verify removal of `isCollectionCreated` storage variable
    ✔ SHOULD deploy contracts
    ✔ SHOULD revert when setting data on non existent collection
    ✔ SHOULD revert when updating non existent collection
    ✔ SHOULD revert when changing meta data of a non existent collection
    ✔ SHOULD revert when freezing non existent collection
    ✔ SHOULD create a collection
    ✔ SHOULD succeed when setting data on an existing collection
    ✔ SHOULD succeed when updating an existing collection
    ✔ SHOULD succeed when changing meta data of an existing collection
    ✔ SHOULD succeed when freezing an existing collection

·----------------------------------------|---------------------------|-------------|-----------------------------·
|          Solc version: 0.8.19          ·  Optimizer enabled: true  ·  Runs: 200  ·  Block limit: 30000000 gas  │
·········································|···························|·············|······························
|  Methods                                                                                                       │
················|························|·············|·············|·············|···············|··············
|  Contract     ·  Method                ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
················|························|·············|·············|·············|···············|··············
|  NextGenCore  ·  changeMetadataView    ·          -  ·          -  ·      62708  ·            1  ·          -  │
················|························|·············|·············|·············|···············|··············
|  NextGenCore  ·  createCollection      ·          -  ·          -  ·     230253  ·            1  ·          -  │
················|························|·············|·············|·············|···············|··············
|  NextGenCore  ·  freezeCollection      ·          -  ·          -  ·      57026  ·            1  ·          -  │
················|························|·············|·············|·············|···············|··············
|  NextGenCore  ·  setCollectionData     ·          -  ·          -  ·     199031  ·            1  ·          -  │
················|························|·············|·············|·············|···············|··············
|  NextGenCore  ·  updateCollectionInfo  ·          -  ·          -  ·      51946  ·            1  ·          -  │
················|························|·············|·············|·············|···············|··············
|  Deployments                           ·                                         ·  % of limit   ·             │
·········································|·············|·············|·············|···············|··············
|  DelegationManagementContract          ·          -  ·          -  ·    4559979  ·       15.2 %  ·          -  │
·········································|·············|·············|·············|···············|··············
|  NextGenAdmins                         ·          -  ·          -  ·     582355  ·        1.9 %  ·          -  │
·········································|·············|·············|·············|···············|··············
|  NextGenCore                           ·          -  ·          -  ·    5490217  ·       18.3 %  ·          -  │
·········································|·············|·············|·············|···············|··············
|  NextGenMinterContract                 ·          -  ·          -  ·    5454331  ·       18.2 %  ·          -  │
·········································|·············|·············|·············|···············|··············
|  NextGenRandomizerNXT                  ·          -  ·          -  ·     634062  ·        2.1 %  ·          -  │
·········································|·············|·············|·············|···············|··············
|  randomPool                            ·          -  ·          -  ·    1024223  ·        3.4 %  ·          -  │
·----------------------------------------|-------------|-------------|-------------|---------------|-------------·

  10 passing (2s)

```

### 2. 976 gas can be saved during creation of collection

| **Method**       | NextGenCore:createCollection() |
|------------------|--------------------------------|
| **Before**       | 252555                         |
| **After**        | 251579 (-976)                  |
| **# Calls**      | 4                              |
| **Test passes?** | Yes                            |

Refactoring the creation of a collection, a critical entry point, into simpler code can save gas:

```diff
    function createCollection(__SNIP__) {
-        collectionInfo[newCollectionIndex].collectionName = _collectionName;
-        collectionInfo[newCollectionIndex].collectionArtist = _collectionArtist;
-        collectionInfo[newCollectionIndex].collectionDescription = _collectionDescription;
-        collectionInfo[newCollectionIndex].collectionWebsite = _collectionWebsite;
-        collectionInfo[newCollectionIndex].collectionLicense = _collectionLicense;
-        collectionInfo[newCollectionIndex].collectionBaseURI = _collectionBaseURI;
-        collectionInfo[newCollectionIndex].collectionLibrary = _collectionLibrary;
-        collectionInfo[newCollectionIndex].collectionScript = _collectionScript;
-        isCollectionCreated[newCollectionIndex] = true;
-        newCollectionIndex = newCollectionIndex + 1;
 
+       isCollectionCreated[newCollectionIndex] = true;
+       collectionInfo[newCollectionIndex++] = collectionInfoStructure(
+           _collectionName,
+           _collectionArtist,
+           _collectionDescription,
+           _collectionWebsite,
+           _collectionLicense,
+           _collectionBaseURI,
+           _collectionLibrary,
+           _collectionScript
+       );
	}
```

[📄 Source: smart-contracts/NextGenCore.sol#L130-L141](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L130-L141)

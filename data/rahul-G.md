## Summary

| **Optimization**                                        | **Gas Saved** |
|---------------------------------------------------------|---------------|
| [G-1] Derive min / max collection bounds dynamically    |         54,522 |
| [G-2] Remove redundant flag for creation of collection  |         34,044 |
| [G-3] Remove duplicate reference to core contract       |         28,517 |
| [G-4] Remove redundant flag for setting up collection   |         26,581 |
| [G-5] Remove duplicate reference to randomizer contract |         22,761 |
| [G-6] Remove redundant external call during airdrop     |          3066 |
| [G-7] Optimise creation of collection                   |           976 |

## Findings
### [G-1] 54,522 gas can be saved by deriving min / max collection bounds dynamically
#### Impact

| **Method**                      | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|---------------------------------|------------|-----------|---------------|------------------|
| NextGenCore:setCollectionData() |     199090 |    154389 |         44701 | Yes              |
| NextGenCore:burn()              |      84270 |     83948 |           322 | Yes              |
| NextGenCore:setFinalSupply()    |      54949 |     49590 |          5359 | Yes              |
| NextGenCore:mint()              |     529382 |    525242 |          4140 | Yes              |

Token min / max index define the valid range of tokens within a collection. There is no need to store them in state variables as these can be derived using `collectionID`  and `collectionTotalSupply` and their values can be read using the getter functions wherever required.
#### Recommendation

**STEP 1:**  Update the getter functions
```diff
    function viewTokensIndexMin(...) {
-       return(collectionAdditionalData[_collectionID].reservedMinTokensIndex);
+       return _collectionID * 10000000000;
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L383-L385](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L383-L385)

```diff
    function viewTokensIndexMax(...) {
-       return(collectionAdditionalData[_collectionID].reservedMaxTokensIndex);
+       return _collectionID * 10000000000 + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
	}
```
[📄 Source: smart-contracts/NextGenCore.sol#L389-L391](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L389-L391)

**STEP 2:** Remove declaration
```diff
    struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
-       uint256 reservedMinTokensIndex;
-       uint256 reservedMaxTokensIndex;
        uint setFinalSupplyTimeAfterMint;
        address randomizerContract;
        IRandomizer randomizer;
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L44-L54](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54)

**STEP 3:** Remove assignment:
```diff
   function setCollectionData(...) {
       __SNIP__
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
            collectionAdditionalData[_collectionID].collectionCirculationSupply = 0;
            collectionAdditionalData[_collectionID].collectionTotalSupply = _collectionTotalSupply;
            collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;
-           collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
-           collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
            wereDataAdded[_collectionID] = true;
        } 
      __SNIP__
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L147-L166](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147-L166)

**STEP 4:** Read value via getter:
```diff
    function burn(uint256 _collectionID, uint256 _tokenId) public {
        __SNIP__
-        require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");
+        require ((_tokenId >= this.viewTokensIndexMin(_collectionID)) && (_tokenId <=  this.viewTokensIndexMax(_collectionID)), "id err");
        __SNIP__
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L204-L209](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L204-L209)

**STEP 5:** Since max value is calculated dynamically, no need to update it on supply change
```diff
    function setFinalSupply(...) {
     __SNIP__
        collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
-       collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L307-L311](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L307-L311)

**STEP 6:** Fetch via  value via getter
```diff
    function getTokenName(...)  {
-       uint256 tok = tokenId - collectionAdditionalData[tokenIdsToCollectionIds[tokenId]].reservedMinTokensIndex;
+       uint256 tok = tokenId - this.viewTokensIndexMin(tokenIdsToCollectionIds[tokenId]);
        __SNIP__
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L361-L364](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L361-L364)
#### POC

![Test Result](https://i.imgur.com/3nYU2eN.png)
##### POC test file
```js
const {
  loadFixture,
  time,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;
const MAX_TOTAL_SUPPLY = 10_000_000_000;

describe("Verify removal of min/max params from storage", async function () {
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

  it("SHOULD create a collection", async function () {
    await contracts.hhCore.createCollection(
      "TestCollection",
      "Artist 1",
      "For testing",
      "www.test.com",
      "CCO",
      "https://ipfs.io/ipfs/hash/",
      "",
      ["desc"]
    );
  });

  it("SHOULD successfully set min / max params for a collection", async function () {
    const SUPPLY = 1000;

    await contracts.hhCore.setCollectionData(
      1,
      signers.owner.address,
      50,
      SUPPLY,
      100
    );

    expect(await contracts.hhCore.viewTokensIndexMin(1)).eq(
      1 * MAX_TOTAL_SUPPLY
    );

    expect(await contracts.hhCore.viewTokensIndexMax(1)).eq(
      1 * MAX_TOTAL_SUPPLY + (SUPPLY - 1)
    );
  });

  it("SHOULD mint token", async function () {
    await contracts.hhCore.addMinterContract(contracts.hhMinter);
    await contracts.hhCore.addRandomizer(1, contracts.hhRandomizer);
    await contracts.hhMinter.setCollectionCosts(
      1, // _collectionID
      0, // _collectionMintCost
      0, // _collectionEndMintCost
      0, // _rate
      0, // _timePeriod
      1, // _salesOptions
      "0xD7ACd2a9FD159E69Bb102A1ca21C9a3e3A5F771B" // delAddress
    );
    await contracts.hhMinter.setCollectionPhases(
      1, // _collectionID
      1696931278, // _allowlistStartTime
      1696931278, // _allowlistEndTime
      1696931278, // _publicStartTime
      1700019791, // _publicEndTime
      "0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870" // _merkleRoot
    );
    await contracts.hhMinter.mint(
      1, // _collectionID
      2, // _numberOfTokens
      0, // _maxAllowance
      '{"tdh": "100"}', // _tokenData
      signers.owner.address, // _mintTo
      ["0x8e3c1713145650ce646f7eccd42c4541ecee8f07040fc1ac36fe071bbfebb870"], // _merkleRoot
      signers.addr1.address, // _delegator
      2 //_varg0
    );
  });

  it("SHOULD burn valid token", async function () {
    let min = await contracts.hhCore.viewTokensIndexMin(1);
    await expect(contracts.hhCore.burn(1, min)).not.to.be.reverted;
    expect(await contracts.hhCore.balanceOf(signers.owner.address)).eq(1);
  });

  it("SHOULD successfully set final supply", async function () {
    await time.increaseTo(1700019900);
    await expect(contracts.hhCore.setFinalSupply(1)).not.to.be.reverted;
  });

    it("SHOULD successfully get token name", async function () {
      let min = await contracts.hhCore.viewTokensIndexMin(1);
      // @audit getTokenName was changed to public for the purpose of unit testing.
      expect(await contracts.hhCore.getTokenName(min + 1n)).to.be.eq(
        "TestCollection #1"
      );
    });
});
```
****

### [G-2] 34,044 gas can be saved by removing redundant flag for creation of collection
#### Impact
| **Method**                         | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|------------------------------------|------------|-----------|---------------|------------------|
| NextGenCore:createCollection()     | 252555     | 230253    |         22302 | Yes              |
| NextGenCore:setCollectionData()    | 199078     | 199031    |            47 | Yes              |
| NextGenCore:updateCollectionInfo() | 51993      | 51946     |            47 | Yes              |
| NextGenCore:changeMetadataView()   | 62755      | 62708     |            47 | Yes              |
| NextGenCore:freezeCollection()     | 57073      | 57026     |            47 | Yes              |

**

| **Deployment** | **Before** | **After** | **Gas Saved** |
|----------------|------------|-----------|---------------|
| NextGenCore    |    5501771 |   5490217 |         11554 |

#### Recommendation
The `NextGenCore.sol` contract uses a storage variable called `isCollectionCreated` to check if a collection has been created. 

Alternatively, the existence of a collection can also be verified by checking if the `collectionID` is less than `newCollectionIndex`, since all existing collections will have a value that is less than `newCollectionIndex`. This optimization saves an expensive storage write during contract creation:

**STEP 1:** Remove the storage variable declaration, and write from `createCollection()`.

```diff
__SNIP__
- mapping (uint256 => bool) private isCollectionCreated;
__SNIP__

    function createCollection(...) {
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

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");

```
[📄 Source: smart-contracts/NextGenCore.sol#L148](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L148)

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false), "Not allowed");

```
[📄 Source: smart-contracts/NextGenCore.sol#L239](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L239)

```diff
- require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
+ require(( _collectionID > 0 && _collectionID < newCollectionIndex) && (collectionFreeze[_collectionID] == false), "Not allowed");

```
[📄 Source: smart-contracts/NextGenCore.sol#L267](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L267)

```diff
- require(isCollectionCreated[_collectionID] == true, "No Col");
+ require((_collectionID > 0 && _collectionID < newCollectionIndex), "No Col");
```
[📄 Source: smart-contracts/NextGenCore.sol#L293](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L293)
#### POC

![Test Result](https://i.imgur.com/efAqtwT.png)
##### POC test file
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

****

### [G-3] 28,517 gas can be saved by removing duplicate reference to core contract
| **Method**                             | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|----------------------------------------|------------|-----------|---------------|------------------|
| NextGenMinterContract:airDropTokens() |     744940 |    742940 |          2000 | Yes              |
| NextGenCore:mint()                     |     404474 |    402474 |          2000 | Yes              |

**

| **Deployment**       | **Before** | **After** | **Gas Saved** |
|----------------------|------------|-----------|---------------|
| NextGenRandomizerNXT |     634074 |    609557 |         24517 |
#### Recommendation
This variable is redundant because the `gencoreContract` variable is already used to store the address of the Gencore contract. The `gencore` variable is only used in the `calculateTokenHash` function, which could be easily updated to use the `gencoreContract` variable instead.

**STEP 1:** Remove deceleration
```diff
contract NextGenRandomizerNXT {

    IXRandoms public randoms;
    INextGenAdmins private adminsContract;
    INextGenCore public gencoreContract;
    // @audit redundant storage variable
-    address gencore;  
}
```

**STEP 2:** Remove assignment from constructor
```diff
    constructor(address _randoms, address _admin, address _gencore) {
        randoms = IXRandoms(_randoms);
        adminsContract = INextGenAdmins(_admin);
-       gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
    }
```

**STEP 3:**  Remove assignment from `updateCoreContract`
```diff
    function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
-       gencore = _gencore;
        gencoreContract = INextGenCore(_gencore);
    }
```

**STEP 4:** Update `calculateTokenHash`
```diff
    // function that calculates the random hash and returns it to the gencore contract
    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
-       require(msg.sender == gencore);
+       require(msg.sender == address(gencoreContract))
        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));
        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
    }
```
#### POC 
This can be verified using existing test suite.

****
### [G-4] 26,581 gas can be saved by removing redundant flag for setting up collection
#### Impact
| **Method**                      | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|---------------------------------|------------|-----------|---------------|------------------|
| NextGenCore:setCollectionData() | 192459     | 170269    |         22190 | Yes              |

**

| **Deployment** | **Before** | **After** | **Gas Saved** |
|----------------|------------|-----------|---------------|
| NextGenCore    |    5501771 |   5497380 |          4391 |
#### Recommendation
The `NextGenCore.sol` contract uses a storage variable called `wereDataAdded` to check if additional data has been added to a collection. 

This flag is set to `true` only if `collectionTotalSupply` is not set for a collection. And so, this value can be derived by checking from `collectionTotalSupply`.

**STEP 1**: Remove declaration
```diff

-  // checks if data on a collection were added
-  mapping (uint256 => bool) private wereDataAdded;

__SNIP__
    function setCollectionData(...) {
		__SNIP__
		// @audit As expected, this can be set only once. Since setting supply cannot be
		//		  set to zero: `_collectionTotalSupply` cannot be zero as
		//        expression-1 will underflow.
        if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
			__SNIP__
			// @audit expression-1
			collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
-           wereDataAdded[_collectionID] = true;
        } 
	    __SNIP__
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L64-L65](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L64-L65)

[📄 Source: smart-contracts/NextGenCore.sol#L157](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L157)

**STEP 2**: The getter function can be refactored to use `collectionTotalSupply` as proxy to check if collection data has been added:
```diff
    function retrievewereDataAdded(uint256 _collectionID) external view returns(bool){
-      return wereDataAdded[_collectionID];
+      return collectionAdditionalData[_collectionID].collectionTotalSupply != 0;
	}

```
[📄 Source: smart-contracts/NextGenCore.sol#L378](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L378)

**STEP 3 (optional)**:  This function appears to check if collection data has been initialized correctly, based on how it is used in the codebase. For example:
```solidity
    function airDropTokens(...)  {
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        __SNIP__
    }  
```
[📄 Source: smart-contracts/MinterContract.sol#L181-L192](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L181-L192)

To make the function's purpose more clear, it could be renamed to something like `hasCollectionDataBeenAdded`.

This change would also have the side effect of highlighting the fact that the codebase relies on the `collectionTotalSupply` to verify if required collection data is present. This gives the team an opportunity to review whether this behavior is desired, or if additional constraints should be added.
#### POC
![Test Result](https://i.imgur.com/Jciv5p0.png)
##### POC test file
```js
const {
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;

describe("verify removal of `wereDataAdded` storage variable", async function () {
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

  it("SHOULD return `false` when `retrievewereDataAdded` is called", async function () {
    await expect(await contracts.hhCore.retrievewereDataAdded(1)).to.be.false;
  });

  it("SHOULD set collection data", async function () {
    await expect(
      contracts.hhCore.setCollectionData(1, signers.owner.address, 50, 100, 100)
    ).not.to.be.reverted;
  });

  it("SHOULD return `true` when `retrievewereDataAdded` is called", async function () {
    expect(await contracts.hhCore.retrievewereDataAdded(1)).to.be.true;
  });
});
```
****
### [G-5] 22,761 gas can be saved by  removing duplicate reference to randomizer contract
#### Impact
| **Method**                             | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|----------------------------------------|------------|-----------|---------------|------------------|
| NextGenCore:addRandomizer()            |      70696 |     53535 |         17161 | Yes              |
| NextGenMinterContract::mint()          |     404474 |    402474 |          2000 | Yes              |
| NextGenMinterContract::airDropTokens() |     744940 |    742940 |          2000 | Yes              |
| NextGenMinterContract::burnToMint()    |     276474 |    274874 |          1600 | Yes              |
#### Recommendation
It is sufficient to only store the address of the randomizer in `collectionAdditonalDataStructure`.  The randomizer contract instance can be referenced by providing this address to `IRandomizer` interface. The gas savings also cascade to critical operations such as `mint` , `burnToMint`, and `airDropTokens`. 

**STEP 1:** Remove duplicate declaration from struct
```diff
    struct collectionAdditonalDataStructure {
        address collectionArtistAddress;
        uint256 maxCollectionPurchases;
        uint256 collectionCirculationSupply;
        uint256 collectionTotalSupply;
        uint256 reservedMinTokensIndex;
        uint256 reservedMaxTokensIndex;
        uint setFinalSupplyTimeAfterMint;
        address randomizerContract;
-       IRandomizer randomizer;
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L44-L54](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54)

**Step 2:** Remove the duplicate assignment
```diff
function addRandomizer(...) {
		__SNIP__        
	collectionAdditionalData[_collectionID].randomizerContract = _randomizerContract;
-   collectionAdditionalData[_collectionID].randomizer = IRandomizer(_randomizerContract);
}
```
[📄 Source: smart-contracts/NextGenCore.sol#L170-L174](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L170-L174)

**Step 3:** Get randomizer instance using `IRandomizer` interface
```diff
    function _mintProcessing(...) internal {
      __SNIP__
-    collectionAdditionalData[_collectionID].randomizer.calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
+    IRandomizer(collectionAdditionalData[_collectionID].randomizerContract).calculateTokenHash(_collectionID, _mintIndex, _saltfun_o);
	 __SNIP__
    }
```
[📄 Source: smart-contracts/NextGenCore.sol#L229](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L229)

Optionally, for ease of understanding struct property could be renamed from `randomizerContract` to `randomizerContractAddress`.
#### POC
![Test result](https://i.imgur.com/g7uU9GK.png)
##### POC test file
POC can be verified using the existing test suite by adding the following test to cover `burnToMint`:
```js
    context("Verify `burnToMint`", async function () {
      it("SHOULD burn existing token and mint a new one", async function () {
        contracts.hhCore.tokenOfOwnerByIndex();
        await contracts.hhMinter.initializeBurn(1, 3, true);
        await expect(
          contracts.hhMinter
            .connect(signers.addr1)
            .burnToMint(1, 10000000000, 3, 2, {
              value: await contracts.hhMinter.getPrice(3),
            })
        ).to.not.be.reverted;

        expect(await contracts.hhCore.ownerOf(30000000002)).to.be.eq(
          signers.addr1.address
        );
      });
    });
```

****

### [G-6]  3066 gas can saved by removing redundant external call during airdrop
#### Impact
| **Method**                             | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|----------------------------------------|------------|-----------|---------------|------------------|
| NextGenMinterContract:airDropTokens() |    1126101 |   1123035 |          3066 | Yes              |
#### Recommendation

```diff
    function airDropTokens(...) {
-       require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
        uint256 collectionTokenMintIndex;
        for (uint256 y=0; y< _recipients.length; y++) {
            collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
            require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");
            }
        }
    }
```
[📄 Source: smart-contracts/MinterContract.sol#L182](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L182) 

The removed line of code checks to see if the collection data has been added. If it has not, the function reverts with the error message "Add data". As discussed in [G-4], `retrievewereDataAdded` returns true only once total supply is added. The function later checks to see if the total tokens to be airdropped is within total supply. If they are not, the function will revert with the error message "No supply". Therefore, the first check for whether the collection data has been added is not necessary.
#### POC
![Test Result](https://i.imgur.com/xNlDPtN.png)
##### POC test file
```js
const {
  loadFixture,
} = require("@nomicfoundation/hardhat-toolbox/network-helpers");
const { expect } = require("chai");
const { ethers } = require("hardhat");
const fixturesDeployment = require("../scripts/fixturesDeployment.js");

let signers;
let contracts;

describe("verify removal of redundant external call from airdropTokens", async function () {
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

  it("SHOULD create a collection and setup collection", async function () {
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

    await contracts.hhCore.addMinterContract(contracts.hhMinter);
    await contracts.hhCore.addRandomizer(1, contracts.hhRandomizer);
  });

  it("SHOULD revert when attempting to airdrop when collection is NOT added", async function () {
    await expect(
      contracts.hhMinter.airDropTokens(
        [signers.addr1.address],
        ["data"],
        [1],
        1,
        [5]
      )
    ).to.be.revertedWith("No supply");
  });
  it("SHOULD successfully airdrop tokens when collection data is added", async function () {
    await contracts.hhCore.setCollectionData(
      1,
      signers.owner.address,
      50,
      500,
      100
    );
    await contracts.hhMinter.airDropTokens(
      [signers.addr1.address],
      ["data"],
      [1],
      1,
      [5]
    );
    expect(await contracts.hhCore.balanceOf(signers.addr1.address)).eq(5);
  });
});

```
****
### [G-7] 976 gas can be saved by optimizing creation of collection

#### Impact
| **Method**                     | **Before** | **After** | **Gas Saved** | **Test Passes?** |
|--------------------------------|------------|-----------|---------------|------------------|
| NextGenCore:createCollection() |     252555 |    251579 |           976 | Yes              |
#### Recommendation
Refactoring the creation of a collection, a critical entry point, into simpler code can save gas:

```diff
    function createCollection(...) {
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
#### POC 
This can be verified using existing test suite.

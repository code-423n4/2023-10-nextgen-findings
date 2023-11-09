# Quality Assurance Report

| QA issues | Issues                                                                                                           | Instances |
|-----------|------------------------------------------------------------------------------------------------------------------|-----------|
| [N-01]    | Variable used only in current contract should be marked with private instead of public visibility                | 7         |
| [N-02]    | Typo in struct name `collectionAdditionalDataStructure`                                                          | 1         |
| [N-03]    | Remove redundant check from `addMinter()` function                                                               | 1         |
| [N-04]    | Incorrect `collectionPrimaryAndSecondaryAddresses` comment should be corrected                                   | 1         |
| [N-05]    | Incorrect comment in function `mintAndAuction()` should be corrected                                             | 1         |
| [L-01]    | Missing check in function `artistSignature()` to ensure parameter `_signature` is not empty                      | 1         |
| [L-02]    | Avoid copy pasting OZ libraries and inheriting from them                                                         | 4         |
| [L-03]    | Missing check in function `setCollectionCosts()` to ensure salesOption is in range [1-3]                         | 1         |
| [L-04]    | Missing check in function `setCollectionPhases()` to ensure allowlist/public end time is greater than start time | 1         |

### Total Non-Critical issues: 11 instances across 5 issues
### Total Low-severity issues: 7 instances across 4 issues
### Total QA issues: 18 instances across 9 issues

## [N-01] Variable used only in current contract should be marked with private instead of public visibility

There are 7 instances of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L26)

Variable `newCollectionIndex` is only used in the [NextGenCore.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L26) contract, thus can be marked with private visibility.
```solidity
File: NextGenCore.sol
26:     uint256 public newCollectionIndex;
```

[Link to instances below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L83C1-L101C52)

There are 6 instances here which can be marked private.
```solidity
File: NextGenCore.sol
083:     mapping (uint256 => uint256) public burnAmount;
084: 
085:     // modify the metadata view
086:     mapping (uint256 => bool) public onchainMetadata; 
087: 
088:     // artist signature per collection
089:     mapping (uint256 => string) public artistsSignatures;
090: 
091:     // tokens additional metadata
092:     mapping (uint256 => string) public tokenData;
099: 
100:     // artist signed
101:     mapping (uint256 => bool) public artistSigned; 
102:
105:     address public minterContract;
```

## [N-02] Typo in struct name `collectionAdditionalDataStructure`

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44)

Correct `collectionAdditonalDataStructure` to `collectionAdditionalDataStructure`. **Note: This is just a typo and does not affect contract logic**
```solidity
File: NextGenCore.sol
45:     struct collectionAdditonalDataStructure {
```

## [N-03] Remove redundant check from `addMinter()` function

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L316)

Check on Line 336 is redundant since even if function admin decides to use malicious minter contract, the malicious contract can implement an isMinterContract() function that always returns true in order to pass this check.
```solidity
File: NextGenCore.sol
335:     function addMinterContract(address _minterContract) public FunctionAdminRequired(this.addMinterContract.selector) { 
336:         require(IMinterContract(_minterContract).isMinterContract() == true, "Contract is not Minter"); 
337:         minterContract = _minterContract;
338:     }
```

## [N-04] Incorrect `collectionPrimaryAndSecondaryAddresses` comment should be corrected

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L83)

Since the keys in the mapping points towards the [collectionPrimaryAddresses struct](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L73), on Line 86 the comment should be use `collectionPrimaryAddresses` struct instead of `collectionPrimaryAndSecondaryAddresses` 
```solidity
File: MinterContract.sol
86:     // mapping of collectionPrimaryAndSecondaryAddresses struct
87:     mapping (uint256 => collectionPrimaryAddresses) private collectionArtistPrimaryAddresses;
```

## [N-05] Incorrect comment in function `mintAndAuction()` should be corrected

There is 1 instance of this:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L293)

Correct "after a day passes" to "after a period passes" since NFTs can be minted period-wise, which may or may not be 1 day.
```solidity
File: MinterContract.sol
308:         // users are able to mint after a day passes
309:         require(tDiff>=1, "1 mint/period");
```

## [L-01] Missing check in function `artistSignature()` to ensure parameter `_signature` is not empty

There is 1 instance of this issue:

[Link to instance](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L257C1-L262C6)

It is possible that the artist of a collection forgets to pass the parameter `_signature`, which would cause the empty signature to be accepted and set. This update would be permanent since artistSigned is set to true on Line 261, which prevents the artist from changing to signature to another string value due to the check on Line 259. 

**Solution: Add a check to ensure that `_signature` is not empty**
```solidity
File: smart-contracts/NextGenCore.sol
257:     function artistSignature(uint256 _collectionID, string memory _signature) public {
258:         require(msg.sender == collectionAdditionalData[_collectionID].collectionArtistAddress, "Only artist");
259:         require(artistSigned[_collectionID] == false, "Already Signed");
260:         artistsSignatures[_collectionID] = _signature;
261:         artistSigned[_collectionID] = true;
262:     }
```

## [L-02] Avoid copy pasting OZ libraries and inheriting from them

**Note: There is more than instance of this issue in the codebase such as copy-paste of Address.sol, Math.sol, Strings.sol etc. All of them are inherited directly. For explanation purposes, I've only included MerkleProof.sol below**

There are 4 instances of this issue:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MerkleProof.sol)

Inherit from openzeppelin dependencies instead of copy pasting contract the [MerkleProof.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MerkleProof.sol) contract. This will ensure all contracts are part of the same version without any breaking modifications which could've occurred during the copy paste. 
```solidity
File: MinterContract.sol
18: import "./MerkleProof.sol"; 
```

## [L-03] Missing check in function `setCollectionCosts()` to ensure salesOption is in range [1-3]

There is 1 instance of this issue:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L163)

Missing check to ensure sales option is in the range [1-3]. There is a chance that the CollectionAdmin forgets to pass the salesOption input (defaulting to 0) or even setting a value outside the range. This could lead to the fixed minting cost option being applied ([as seen here](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L564C11-L567C10)) instead of the expected sales options (2 or 3). 
```solidity
File: MinterContract.sol
159:     function setCollectionCosts(uint256 _collectionID, uint256 _collectionMintCost, uint256 _collectionEndMintCost, uint256 _rate, uint256 _timePeriod, uint8 _salesOption, address _delAddress) public CollectionAdminRequired(_collectionID, this.setCollectionCosts.selector) {
160:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data"); 
161:         collectionPhases[_collectionID].collectionMintCost = _collectionMintCost;
162:         collectionPhases[_collectionID].collectionEndMintCost = _collectionEndMintCost;
163:         collectionPhases[_collectionID].rate = _rate;
164:         collectionPhases[_collectionID].timePeriod = _timePeriod;
165:         collectionPhases[_collectionID].salesOption = _salesOption;
166:         collectionPhases[_collectionID].delAddress = _delAddress;
167:         setMintingCosts[_collectionID] = true;
168:     }
```

## [L-04] Missing check in function `setCollectionPhases()` to ensure allowlist/public end time is greater than start time

There is 1 instance of this issue:

[Link to instance below](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L172C1-L176C72)

Missing check to see if `_allowlistEndTime` and `_publicEndTime` are greater than `_allowlistStartTime` and `_publicStartTime` respectively.
```solidity
File: MinterContract.sol
172:     function setCollectionPhases(uint256 _collectionID, uint _allowlistStartTime, uint _allowlistEndTime, uint _publicStartTime, uint _publicEndTime, bytes32 _merkleRoot) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
173:         require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
174:         collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime;
175:         collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
176:         collectionPhases[_collectionID].merkleRoot = _merkleRoot;
177:         collectionPhases[_collectionID].publicStartTime = _publicStartTime;
178:         collectionPhases[_collectionID].publicEndTime = _publicEndTime;
179:     }
```
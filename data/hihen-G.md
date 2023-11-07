# Gas Report

## Summary

Total **36 instances** over **7 issues**with **340334** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G&#x2011;01]](#g01-inline-modifiers-that-are-only-used-once-to-save-gas) | Inline `modifier`s that are only used once to save gas | 1 | - |
| [[G&#x2011;02]](#g02-arrayindex--amount-is-cheaper-than-arrayindex--arrayindex--amount-or-related-variants) | `array[index] += amount` is cheaper than `array[index] = array[index] + amount` (or related variants) | 8 | 304 |
| [[G&#x2011;03]](#g03-empty-code-blocks-can-be-removed-to-save-gas) | Empty code blocks can be removed to save gas | 2 | - |
| [[G&#x2011;04]](#g04-using-assembly-to-check-for-zero-can-save-gas) | Using assembly to check for zero can save gas | 5 | 30 |
| [[G&#x2011;05]](#g05-contracts-can-use-fewer-storage-slots-by-truncating-state-variables) | Contracts can use fewer storage slots by truncating state variables | 1 | 20000 |
| [[G&#x2011;06]](#g06-structs-can-be-packed-into-fewer-storage-slots-by-truncating-fields) | Structs can be packed into fewer storage slots by truncating fields | 6 | 320000 |
| [[G&#x2011;07]](#g07-refactor-duplicated-requirerevert-checks-to-save-gas) | Refactor duplicated `require()`/`revert()` checks to save gas | 13 | - |

## Gas Optimizations

### [G&#x2011;01] Inline `modifier`s that are only used once to save gas

Inline `modifier`s that are only used once can save gas.

There is 1 instance:

- *AuctionDemo.sol* ( [31](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L31) ):

```solidity
31:     modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
```

### [G&#x2011;02] `array[index] += amount` is cheaper than `array[index] = array[index] + amount` (or related variants)

When updating a value in an array with arithmetic, using `array[index] += amount` is cheaper than `array[index] = array[index] + amount`.
This is because you avoid an additional `mload` when the array is stored in memory, and an `sload` when the array is stored in storage.
This can be applied for any arithmetic operation including `+=`, `-=`,`/=`,`*=`,`^=`,`&=`, `%=`, `<<=`,`>>=`, and `>>>=`.
This optimization can be particularly significant if the pattern occurs during a loop.

*Saves 28 gas for a storage array, 38 for a memory array*

There are 8 instances:

- *MinterContract.sol* ( [238](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L238), [271](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L271), [364](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L364) ):

```solidity
238:         collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;

271:         collectionTotalAmount[_mintCollectionID] = collectionTotalAmount[_mintCollectionID] + msg.value;

364:         collectionTotalAmount[col] = collectionTotalAmount[col] + msg.value;
```

- *NextGenCore.sol* ( [183](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L183), [195](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L195), [197](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L197), [208](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L208), [221](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L221) ):

```solidity
183:             tokensAirdropPerAddress[_collectionID][_recipient] = tokensAirdropPerAddress[_collectionID][_recipient] + 1;

195:                 tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;

197:                 tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;

208:         burnAmount[_collectionID] = burnAmount[_collectionID] + 1;

221:             burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;
```

### [G&#x2011;03] Empty code blocks can be removed to save gas

The following empty code blocks can be removed or refactored to save gas.

There are 2 instances:

- *AuctionDemo.sol* ( [118](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L118), [141](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L141) ):

```solidity
118:             } else {}

141:             } else {}
```

### [G&#x2011;04] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

There are 5 instances:

- *MinterContract.sol* ( [242](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L242), [285](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L285), [549](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L549) ):

```solidity
242:             if (lastMintDate[col] == 0) {

285:         if (lastMintDate[_collectionID] == 0) {

549:             if (collectionPhases[_collectionId].rate == 0) {
```

- *NextGenCore.sol* ( [149](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L149) ):

```solidity
149:         if (collectionAdditionalData[_collectionID].collectionTotalSupply == 0) {
```

- *XRandoms.sol* ( [28](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L28) ):

```solidity
28:         if (id==0) {
```

### [G&#x2011;05] Contracts can use fewer storage slots by truncating state variables

Some state variables can be safely modified so that the contract uses fewer storage slots. Each saved slot can avoid an extra Gsset (**20000 gas**) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There is 1 instance:

- *NextGenCore.sol* ( [22](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L22) ):

```solidity
/// @audit Truncate `newCollectionIndex` to `uint64`
/// @audit Fewer storage usage (19 slots -> 18 slots):
///        32 B: mapping (uint256 => collectionInfoStructure) collectionInfo
///        32 B: mapping (uint256 => collectionAdditonalDataStructure) collectionAdditionalData
///        32 B: mapping (uint256 => bool) isCollectionCreated
///        32 B: mapping (uint256 => bool) wereDataAdded
///        32 B: mapping (uint256 => uint256) tokenIdsToCollectionIds
///        32 B: mapping(uint256 => bytes32) tokenToHash
///        32 B: mapping (uint256 => mapping (address => uint256)) tokensMintedPerAddress
///        32 B: mapping (uint256 => mapping (address => uint256)) tokensMintedAllowlistAddress
///        32 B: mapping (uint256 => mapping (address => uint256)) tokensAirdropPerAddress
///        32 B: mapping (uint256 => uint256) burnAmount
///        32 B: mapping (uint256 => bool) onchainMetadata
///        32 B: mapping (uint256 => string) artistsSignatures
///        32 B: mapping (uint256 => string) tokenData
///        32 B: mapping (uint256 => string[2]) tokenImageAndAttributes
///        32 B: mapping (uint256 => bool) collectionFreeze
///        32 B: mapping (uint256 => bool) artistSigned
///        20 B: INextGenAdmins adminsContract
///        4 B: uint64 newCollectionIndex
///        20 B: address minterContract
22: contract NextGenCore is ERC721Enumerable, Ownable, ERC2981 {
```

### [G&#x2011;06] Structs can be packed into fewer storage slots by truncating fields

Some struct fields  can be safely modified so that the struct variable uses fewer storage slots. Each saved slot can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

<details>
<summary>There are 6 instances (click to show):</summary>

- *MinterContract.sol* ( [44-56](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L44-L56), [63-66](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L63-L66), [73-81](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L73-L81), [88-91](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L88-L91), [98-106](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L98-L106) ):

```solidity
/// @audit Truncate `allowlistStartTime` to `uint32`
/// @audit Truncate `allowlistEndTime` to `uint32`
/// @audit Truncate `publicStartTime` to `uint32`
/// @audit Truncate `publicEndTime` to `uint32`
/// @audit Truncate `timePeriod` to `uint32`
/// @audit Truncate `rate` to `uint32`
/// @audit Fewer storage usage (10 slots -> 5 slots):
///        32 B: bytes32 merkleRoot
///        32 B: uint256 collectionMintCost
///        32 B: uint256 collectionEndMintCost
///        20 B: address delAddress
///        4 B: uint32 allowlistStartTime
///        4 B: uint32 allowlistEndTime
///        4 B: uint32 publicStartTime
///        4 B: uint32 publicEndTime
///        4 B: uint32 timePeriod
///        4 B: uint32 rate
///        1 B: uint8 salesOption
44:     struct collectionPhasesDataStructure {
45:         uint allowlistStartTime;
46:         uint allowlistEndTime;
47:         uint publicStartTime;
48:         uint publicEndTime;
49:         bytes32 merkleRoot;
50:         uint256 collectionMintCost;
51:         uint256 collectionEndMintCost;
52:         uint256 timePeriod;
53:         uint256 rate;
54:         uint8 salesOption;
55:         address delAddress;
56:     }

/// @audit Truncate `artistPercentage` to `uint32`
/// @audit Truncate `teamPercentage` to `uint32`
/// @audit Fewer storage usage (2 slots -> 1 slots):
///        4 B: uint32 artistPercentage
///        4 B: uint32 teamPercentage
63:     struct royaltiesPrimarySplits {
64:         uint256 artistPercentage;
65:         uint256 teamPercentage;
66:     }

/// @audit Truncate `add1Percentage` to `uint32`
/// @audit Truncate `add2Percentage` to `uint32`
/// @audit Truncate `add3Percentage` to `uint32`
/// @audit Fewer storage usage (7 slots -> 3 slots):
///        20 B: address primaryAdd1
///        4 B: uint32 add1Percentage
///        4 B: uint32 add2Percentage
///        4 B: uint32 add3Percentage
///        20 B: address primaryAdd2
///        1 B: bool status
///        20 B: address primaryAdd3
73:     struct collectionPrimaryAddresses {
74:         address primaryAdd1;
75:         address primaryAdd2;
76:         address primaryAdd3;
77:         uint256 add1Percentage;
78:         uint256 add2Percentage;
79:         uint256 add3Percentage;
80:         bool status;
81:     }

/// @audit Truncate `artistPercentage` to `uint32`
/// @audit Truncate `teamPercentage` to `uint32`
/// @audit Fewer storage usage (2 slots -> 1 slots):
///        4 B: uint32 artistPercentage
///        4 B: uint32 teamPercentage
88:     struct royaltiesSecondarySplits {
89:         uint256 artistPercentage;
90:         uint256 teamPercentage;
91:     }

/// @audit Truncate `add1Percentage` to `uint32`
/// @audit Truncate `add2Percentage` to `uint32`
/// @audit Truncate `add3Percentage` to `uint32`
/// @audit Fewer storage usage (7 slots -> 3 slots):
///        20 B: address secondaryAdd1
///        4 B: uint32 add1Percentage
///        4 B: uint32 add2Percentage
///        4 B: uint32 add3Percentage
///        20 B: address secondaryAdd2
///        1 B: bool status
///        20 B: address secondaryAdd3
98:     struct collectionSecondaryAddresses {
99:         address secondaryAdd1;
100:         address secondaryAdd2;
101:         address secondaryAdd3;
102:         uint256 add1Percentage;
103:         uint256 add2Percentage;
104:         uint256 add3Percentage;
105:         bool status;
106:     }
```

- *NextGenCore.sol* ( [44-54](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L44-L54) ):

```solidity
/// @audit Truncate `setFinalSupplyTimeAfterMint` to `uint32`
/// @audit Fewer storage usage (9 slots -> 8 slots):
///        32 B: uint256 maxCollectionPurchases
///        32 B: uint256 collectionCirculationSupply
///        32 B: uint256 collectionTotalSupply
///        32 B: uint256 reservedMinTokensIndex
///        32 B: uint256 reservedMaxTokensIndex
///        20 B: address collectionArtistAddress
///        4 B: uint32 setFinalSupplyTimeAfterMint
///        20 B: address randomizerContract
///        20 B: IRandomizer randomizer
44:     struct collectionAdditonalDataStructure {
45:         address collectionArtistAddress;
46:         uint256 maxCollectionPurchases;
47:         uint256 collectionCirculationSupply;
48:         uint256 collectionTotalSupply;
49:         uint256 reservedMinTokensIndex;
50:         uint256 reservedMaxTokensIndex;
51:         uint setFinalSupplyTimeAfterMint;
52:         address randomizerContract;
53:         IRandomizer randomizer;
54:     }
```

</details>

### [G&#x2011;07] Refactor duplicated `require()`/`revert()` checks to save gas

Duplicate `require()`/`revert()` checks can be refactored into a modifier or function, saving deployment costs.

<details>
<summary>There are 13 instances (click to show):</summary>

- *AuctionDemo.sol* ( [125](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L125) ):

```solidity
/// @audit Duplicated on line 135
125:         require(block.timestamp <= minter.getAuctionEndTime(_tokenid), "Auction ended");
```

- *MinterContract.sol* ( [158](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L158), [171](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L171), [186](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L186), [211](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L211), [220](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L220), [232](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L232) ):

```solidity
/// @audit Duplicated on line 182, 277
158:         require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");

/// @audit Duplicated on line 197
171:         require(setMintingCosts[_collectionID] == true, "Set Minting Costs");

/// @audit Duplicated on line 280
186:             require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

/// @audit Duplicated on line 337
211:                 require(isAllowedToMint == true, "No delegation");

/// @audit Duplicated on line 350
220:             require(MerkleProof.verifyCalldata(merkleProof, collectionPhases[col].merkleRoot, node), 'invalid proof');

/// @audit Duplicated on line 360
232:         require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```

- *NextGenCore.sol* ( [179](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L179), [239](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L239) ):

```solidity
/// @audit Duplicated on line 190, 214
179:         require(msg.sender == minterContract, "Caller is not the Minter Contract");

/// @audit Duplicated on line 267
239:         require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```

- *RandomizerRNG.sol* ( [41](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L41) ):

```solidity
/// @audit Duplicated on line 54
41:         require(msg.sender == gencore);
```

- *RandomizerVRF.sol* ( [53](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L53) ):

```solidity
/// @audit Duplicated on line 72
53:         require(msg.sender == gencore);
```

</details>

## Low
### [L-01] Incorrect index to value mapping in wordsList.
The id range should be [0, 100) according to ```randomWord()```. Actually, the id can be used directly as the index of the ```wordsList```. We just need ```return wordsList[id];``` for simplicity and readability.

```solidity
File: smart-contracts/XRandoms.sol

// origin:
27:        // returns a word based on index
28:        if (id==0) {
29:            return wordsList[id];
30:        } else {
31:            return wordsList[id - 1];
32:        }

// optimize:
27:        // returns a word based on index
28:        return wordsList[id];
```
[[L27-L32]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L27-L32)


## Non-Critical
### [N-01] Function mutability can be restricted to pure.
Change function mutability to pure if it does not access or modify the state.
```solidity
File: smart-contracts/NextGenAdmins.sol
83:    function isAdminContract() external view returns (bool) {
84:        return true;
85:    }

File: smart-contracts/MinterContract.sol
506:    function isMinterContract() external view returns (bool) {
507:        return true;
508:    }

File: smart-contracts/RandomizerNXT.sol
62:    function isRandomizerContract() external view returns (bool) {
63:        return true;
64:    }

File: smart-contracts/RandomizerRNG.sol
89:    function isRandomizerContract() external view returns (bool) {
90:        return true;
91:    }

File: smart-contracts/RandomizerVRF.sol
105:    function isRandomizerContract() external view returns (bool) {
106:        return true;
107:    }
```
[[L83-L85]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L83-L85) | [[L506-L508]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L506-L508) | [[L62-L64]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L62-L64) | [[L89-L91]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L89-L91) | [[L105-L107]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L105-L107)

### [N-02] Exact same struct with different names.
```royaltiesPrimarySplits``` and ```royaltiesSecondarySplits``` have the same struct definition. Consider define it as ```royaltiesSplits```. The same goes for ```collectionPrimaryAddresses``` and ```collectionSecondaryAddresses```.

```solidity
File: smart-contracts/MinterContract.sol

 63:    struct royaltiesPrimarySplits {
 64:        uint256 artistPercentage;
 65:        uint256 teamPercentage;
 66:    }

 73:    struct collectionPrimaryAddresses {
 74:        address primaryAdd1;
 75:        address primaryAdd2;
 76:        address primaryAdd3;
 77:        uint256 add1Percentage;
 78:        uint256 add2Percentage;
 79:        uint256 add3Percentage;
 80:        bool status;
 81:    }

 88:    struct royaltiesSecondarySplits {
 89:        uint256 artistPercentage;
 90:        uint256 teamPercentage;
 91:    }

 98:    struct collectionSecondaryAddresses {
 99:        address secondaryAdd1;
100:        address secondaryAdd2;
101:        address secondaryAdd3;
102:        uint256 add1Percentage;
103:        uint256 add2Percentage;
104:        uint256 add3Percentage;
105:        bool status;
106:    }
```
[[L63-L106]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L63-L106)

### [N-03] Using address(0) for null address.
Using ```address(0)``` instead of ```0x0000000000000000000000000000000000000000``` for null address comparation.
```solidity
File: smart-contracts/MinterContract.sol
205:            if (_delegator != 0x0000000000000000000000000000000000000000) {
```
[[205]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L205)

### [N-04] Redundant require statement.
The require statement in L223 is redundant and not accurate enough. All we need is the L224.
```solidity
File: smart-contracts/MinterContract.sol
223:            require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
224:            require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
```
[[L223-L224]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L223-L224)

### [N-05] Exclude end time in time period comparison.
As for comparing with end time, use "<" instead of "<=", avoiding a time point belong to both allowlist phase and public phase.
```solidity
File: smart-contracts/MinterContract.sol
202:        if (block.timestamp >= collectionPhases[col].allowlistStartTime && block.timestamp <= collectionPhases[col].allowlistEndTime) {
            ...
221:        } else if (block.timestamp >= collectionPhases[col].publicStartTime && block.timestamp <= collectionPhases[col].publicEndTime) {
            ...
        }
```
[[L202-L221]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L202-L221)

### [N-06] Identical require statements exist both in caller and callee function.
```require(msg.sender == gencore);``` will be checked in ```requestRandomWords``` (callee), there is no need to check it in ```calculateTokenHash``` (caller).
```solidity
File: smart-contracts/RandomizerRNG.sol
40:    function requestRandomWords(uint256 tokenid, uint256 _ethRequired) public payable {
41:        require(msg.sender == gencore);
           ...
46:    }

53:    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
54:        require(msg.sender == gencore);
55:        tokenIdToCollection[_mintIndex] = _collectionID;
56:        requestRandomWords(_mintIndex, ethRequired);
57:    }

File: smart-contracts/RandomizerVRF.sol
52:    function requestRandomWords(uint256 tokenid) public {
53:        require(msg.sender == gencore);
           ...
63:    }

71:    function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
72:        require(msg.sender == gencore);
73:        tokenIdToCollection[_mintIndex] = _collectionID;
74:        requestRandomWords(_mintIndex);
75:    }
```
[[L40-L57]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L40-L57) | [[L52-L75]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L52-L75)

### [N-07] ```airDropTokens``` should be ```airDropToken```.
This function only airdrop one single token, the function name should be ```airDropToken```, without the ending "s".
```solidity
File: smart-contracts/NextGenCore.sol
178:    function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external
```
[[L178]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178)

### [N-08] else/else-if is not needed if all branches have return statements.
If "return" statement resides in "if" branch, there is no need to use "else-if" or "else". Use another "if" for "else-if".
```
File: smart-contracts/NextGenCore.sol
345:        if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] != 0x0000000000000000000000000000000000000000000000000000000000000000) {
346:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
347:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
348:        } else if (onchainMetadata[tokenIdsToCollectionIds[tokenId]] == false && tokenToHash[tokenId] == 0x0000000000000000000000000000000000000000000000000000000000000000) {
349:            string memory baseURI = collectionInfo[tokenIdsToCollectionIds[tokenId]].collectionBaseURI;
350:            return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, "pending")) : "";
351:        }
352:        else {
353:            string memory b64 = ...;
354:            string memory _uri = ...;
355:            return _uri;
356:        }

File: smart-contracts/MinterContract.sol
532:        if (collectionPhases[_collectionId].salesOption == 3) {
                ...
535:            if (collectionPhases[_collectionId].rate > 0) {
536:                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
537:            } else {
538:                return collectionPhases[_collectionId].collectionMintCost;
539:            }
540:        } else if (collectionPhases[_collectionId].salesOption == 2 && block.timestamp > collectionPhases[_collectionId].allowlistStartTime && block.timestamp < collectionPhases[_collectionId].publicEndTime){
                ...
559:            if (price - decreaserate > collectionPhases[_collectionId].collectionEndMintCost) {
560:                return price - decreaserate; 
561:            } else {
562:                return collectionPhases[_collectionId].collectionEndMintCost;
563:            }
564:        } else {
565:            // fixed price
566:            return collectionPhases[_collectionId].collectionMintCost;
567:        }
```
[[L345-L356]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L345-L356) | [[L532-L567]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L532-L567)

### [N-09] Use ```reservedMinTokensIndex``` instead of ```_collectionID * 10000000000```.
In ```setFinalSupply```, use ```reservedMinTokensIndex``` instead of ```_collectionID * 10000000000``` to calculate ```reservedMaxTokensIndex``` for readability.
```solidity
File: smart-contracts/NextGenCore.sol
310:        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
```
[[L307-L311]](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L310)
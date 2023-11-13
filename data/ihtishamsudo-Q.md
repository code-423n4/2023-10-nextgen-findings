## Executive Summary 

Following Quality Assurance Reports Are The Notes, Refactoring Recommendations, And Some Low Severity bugs Found During the Auditing process.


## Table of Contents
* 1. [L- NextGenCore is using Deperecated ERC721 Enumerable Library](#L-NextGenCoreisusingDeperecatedERC721EnumerableLibrary)
* 2. [L - Predictable Randomness Vulnerability](#L-PredictableRandomnessVulnerability)
* 3. [L - Use of Bool Instead Of Require](#L-UseOfBoolInsteadOfRequire)
* 4. [L - Don't use hardcoded values If plan to upgrade contract in future](#L-DontusehardcodedvaluesIfplantoupgradecontractinfuture)
* 5. [L - Missing Check If Collection Exist or Not](#L-MissingCheckIfCollectionExistorNot)
* 6. [L - As protocol relies heavily on admins, single-step ownership transfer pattern is dangerous ](#L-singlestepownershiptransferpatternisdangerous)
* 7. [L - RandomizerNXT should use ownable like other VRF, RNG Randomizer](#L-RandomizerNXTshoulduseownablelikeotherVRF,RNGRandomizer)
* 8. [NC - Use one standard for uint256](#NC-Useonestandardforuint256)
* 9. [NC - Use propoer header alignment](#NC-useproperheaderalignment)
* 10. [NC - Deprecated Documentation About Primary & Secondry Split](#NC-Deprecatedprimary&secondarySplit)

## 1. <a name='L-NextGenCoreisusingDeperecatedERC721EnumerableLibrary'></a> L - NextGenCore is using Deperecated ERC721 Enumerable Library
[NextGenCore.sol#L22](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenCore.sol#L22)

Current NextGen core is using ERC721 Enumerable Library which is deprecated and many of the functions has been removed and updated as the last update of Openzepplin 5.0.0.
```solidity
    // SPDX-License-Identifier: MIT
    // OpenZeppelin Contracts (last updated v4.8.0) (token/ERC721/extensions/ERC721Enumerable.sol)
```
has many functions updated and removed as highlighted below 
```solidity 
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 firstTokenId,
        uint256 batchSize
    ) internal virtual override {
        //exisiting code
    }
```
In new update of OZ 5.0.0, it is stated 
ERC20, ERC721, ERC1155: Deleted _beforeTokenTransfer and _afterTokenTransfer hooks, added a new internal _update function for customizations, and refactored all extensions using those hooks to use _update instead. (#3838, #3876, #4377)

## 2. <a name='L-PredictableRandomnessVulnerability'></a> L - Predictable Randomness Vulnerability
[RandomizerNXT.sol#L57](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/RandomizerNXT.sol#L57)

`RandomizerNXT.sol` is using block.hash, block.timestamp for calculating random hash which is not a good practice as it is predictable and can be manipulated by miners. Usage of block.hash, block.timestamp is often discouraged by not recommended as source of randomness.
```solidity
     function calculateTokenHash(uint256 _collectionID, uint256 _mintIndex, uint256 _saltfun_o) public {
        require(msg.sender == gencore); //@using blockhash to calculate hash 
        bytes32 hash = keccak256(abi.encodePacked(_mintIndex, blockhash(block.number - 1), randoms.randomNumber(), randoms.randomWord()));  //@audit deprecated method ref DeFiVulnLabs 
        gencoreContract.setTokenHash(_collectionID, _mintIndex, hash);
    }
```
`Mitigation` : Don't use block.hash, block.timestamp as source of randomness.
`Ref` : https://solidity-by-example.org/hacks/randomness/

## 3. <a name='L-UseOfBoolInsteadOfRequire'></a> L - Use of Bool Instead Of Require
[MinterContract.sol#L434-L438](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L434-L438)

The current implementation in the code uses a bool variable to capture the success of external calls, specifically in the following lines for sending payments:
```solidity
(bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
(bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
(bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
(bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
(bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```

It is advised to use the `require` statement instead of relying solely on the bool variable to ensure the success of calls.

`Mitigation` : 
```solidity
require(success1, "Transfer failed"); //same for every bool call
```

## 4. <a name='L-DontusehardcodedvaluesIfplantoupgradecontractinfuture'></a> L - Don't use hardcoded values If plan to upgrade contract in future

[MinterContract.sol#L448](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/MinterContract.sol#L448)
[NextGenCore.sol#L111](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenCore.sol#L111)

In minter contract there are functions to update Core, admin contracts but in core contracts hardcoded values are being used for setting default royality.

```solidity
    function updateCoreContract(address _gencore) public FunctionAdminRequired(this.updateCoreContract.selector) { 
        gencore = INextGenCore(_gencore);
    }
```

```solidity
    constructor(string memory name, string memory symbol, address _adminsContract) ERC721(name, symbol) {
        adminsContract = INextGenAdmins(_adminsContract);
        newCollectionIndex = newCollectionIndex + 1;
        _setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690); //@audit Hardcoded Value 
    }
```
## 5. <a name='L-MissingCheckIfCollectionExistorNot'></a> L - Missing Check If Collection Exist or Not
[NextGenCore.sol#L426](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/NextGenCore.sol#L426)

NextGen Gitbook docuementation says for getter function to retrieve collection info collection must exist i.e,

```solidity 
Get Collection Info
The retrieveCollectionInfo(..) function returns the full info of a collection.
/**
  * @dev Retrieve a collection's information.
  * @param _collectionID Refers to the specific collection for which the info will be returned.
*/

function retrieveCollectionInfo(
  uint256 _collectionID
) public view returns (string memory, sring memory, string memory, string memory, string memory, string memory) {
  return (collectionName, collectionArtist, collectionDescription, collectionWebsite, collectionLicense, collectionBaseURI);
}
Notes:
The collection should exist.
```

but no check in actual if collection exist or not 

```solidity
  function retrieveCollectionInfo(uint256 _collectionID) public view returns(string memory, string memory, string memory, string memory, string memory, string memory){
        return (collectionInfo[_collectionID].collectionName, collectionInfo[_collectionID].collectionArtist, collectionInfo[_collectionID].collectionDescription, collectionInfo[_collectionID].collectionWebsite, collectionInfo[_collectionID].collectionLicense, collectionInfo[_collectionID].collectionBaseURI);
    }
```
This check is not implemented in any getter function and needs to be implmeneted in other getter functions as well as per NextGen documentation.

## 6. <a name='L-singlestepownershiptransferpatternisdangerous'></a> L - As protocol relies heavily on admins, single-step ownership transfer pattern is dangerous

[NextGenAdmins.sol#L15](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L15)

```solidity
contract NextGenAdmins is Ownable{
    //exisitng code
}
```
Inheriting from OpenZeppelin's Ownable contract means you are using a single-step ownership transfer pattern.
The better way to do this is to use a two-step ownership transfer approach, where the new owner should first claim its new rights before they are transferred.

`Mitigation` : Use  OpenZeppelin's `Ownable2Step` instead of `Ownable`

## 7. <a name='L-RandomizerNXTshoulduseownablelikeotherVRF,RNGRandomizer'></a> L - RandomizerNXT should use ownable like other VRF, RNG Randomizer

[RandomizerNXT.sol#L14](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L18)

Other Randomizer i.e, VRF, RNG Using Ownable contract but RandomizerNXT is not using Ownable contract.
It is only importing but not implementing Ownable contract.
```solidity
contract NextGenRandomizerNXT {
    //existing code 
}
```
`Mitigation` : Implement `Ownable` (ownable2step according to my L - 6) contract in `RandomizerNXT` contract as well.

## 8. <a name='NC-Useonestandardforuint256'></a> NC - Use one standard for uint256

[MinterContract.sol#L44](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L44) 

Througout the whole codebase both standards used for uint i.e, uint and uint256. It is better to use one standard for uint i.e, uint256.

```solidity
struct collectionPhasesDataStructure {
        uint allowlistStartTime;
        //
        //
        uint256 collectionMintCost;
        uint256 collectionEndMintCost;
        uint256 timePeriod;
        uint256 rate;
        //
    }
```
`Mitigation` : Use one standard for uint i.e, uint256 or uint not both

## 9. <a name='NC-useproperheaderalignment'></a> NC - Use propoer header alignment
[NextGenCore.sol#L333](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L333)

Protocol code style uses a short, and somehwat asymetrical, form of comment header separation. To improve readability, change headers to a more visible version.

For instance : [This Header](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L333) 
    // Retrieve Functions

can be replaced with [transmissions11/headers](https://github.com/transmissions11/headers)


   /*//////////////////////////////////////////////////////////////

                           Retrieve Functions

   //////////////////////////////////////////////////////////////*/

## 10. <a name='NC-Deprecatedprimary&secondarySplit'></a> NC - Deprecated Documentation About Primary & Secondry Split

According to the NextGen Gitbook deocumentation there are two separate functions for setting primary split & secondary split but in actual there is only one function for setting both primary & secondary split.

```solidity


    function setPrimaryAndSecondarySplits(uint256 _collectionID, uint256 _artistPrSplit, uint256 _teamPrSplit, uint256 _artistSecSplit, uint256 _teamSecSplit) public FunctionAdminRequired(this.setPrimaryAndSecondarySplits.selector) {
        require(_artistPrSplit + _teamPrSplit == 100, "splits need to be 100%");
        require(_artistSecSplit + _teamSecSplit == 100, "splits need to be 100%");
        collectionRoyaltiesPrimarySplits[_collectionID].artistPercentage = _artistPrSplit;
        collectionRoyaltiesPrimarySplits[_collectionID].teamPercentage = _teamPrSplit;
        collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage = _artistSecSplit;
        collectionRoyaltiesSecondarySplits[_collectionID].teamPercentage = _teamSecSplit;
    }

```

`Gitbook Docs` : https://seize-io.gitbook.io/nextgen/nextgen-smart-contracts/minter

`Mitigation` : Update Gitbook Docs according to actual implementation.
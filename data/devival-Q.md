## Low Severity Issues
### L-01: Missing checks for `_setFinalSupplyTimeAfterMint` param in the `setCollectionData` would make the collection idle. 
Recommendation: add `require(_setFinalSupplyTimeAfterMint > block.timestamp)` in `setCollectionData` of NextGenCore.sol: [147](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147)

### L-02: Missing checks for `isAdminContract()` in `RandomizerNXT.updateAdminsContract`
 
All main contracts such as NextGenCore, MinterContract, RandomizerRNG, and RandomizerVRF have a check if the new admins' contract is an admin contract. However, the RandomizerNXT lacks this check. This would allow admins to submit irrelevant contract as admin contract in RandomizerNXT. 

Recommendation: Replace
```solidity
function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {
        adminsContract = INextGenAdmins(_admin);
    }
```
[RandomizerNXT.sol#L45](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L45)
with:
```solidity
function updateAdminsContract(address _admin) public FunctionAdminRequired(this.updateAdminsContract.selector) {
        require(INextGenAdmins(_admin).isAdminContract() == true, "Contract is not Admin");
        adminsContract = INextGenAdmins(_admin);
    }
```

### L-03: Missing checks for the `_setFinalSupplyTimeAfterMint` param in `setCollectionData` would make the collection idle. 
Recommendation: add `require(_setFinalSupplyTimeAfterMint > block.timestamp)` in `setCollectionData` of NextGenCore.sol: [147](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147)

### L-04: `totalSupplyOfCollection` and `collectionTotalSupply` return different values

***Impact***
According to the docs, the `retrieveCollectionAdditionalData()` function returns the additional data that were set for a collection, including `collectionTotalSupply`. Another one, the `totalSupplyOfCollection()` function, returns the total token supply of a collection (`collectionCirculationSupply - burnAmount`).

As a consequence, when there is a difference between `collectionCirculationSupply` and `collectionTotalSupply`, or if some tokens are burned, the `totalSupplyOfCollection` value would decrease while the `collectionAdditionalData[_collectionID].collectionTotalSupply` would remain the same. In this case, the `retrieveCollectionAdditionalData()` function would not provide up-to-date information to the users and leak its value. 

***Proof of Concept***
1. Alice creates a new collection and sets `collectionTotalSupply` to 10
2. Bobby mints 5 tokens:
3. Bobby wants to check the total supply of collection with `retrieveCollectionAdditionalData(_collectionID).collectionTotalSupply` but would see 10 as a return value instead of 5, which he would see in `totalSupplyOfCollection`; this would make the user confused.

`retrieveCollectionAdditionalData` and `totalSupplyOfCollection` functions:
```
File: smart-contracts/NextGenCore.sol

    function retrieveCollectionAdditionalData(uint256 _collectionID) public view returns(address, uint256, uint256, uint256, uint, address){
        return (collectionAdditionalData[_collectionID].collectionArtistAddress, collectionAdditionalData[_collectionID].maxCollectionPurchases, collectionAdditionalData[_collectionID].collectionCirculationSupply, collectionAdditionalData[_collectionID].collectionTotalSupply, collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, collectionAdditionalData[_collectionID].randomizerContract);
    }
    // .. other code
    function totalSupplyOfCollection(uint256 _collectionID) public view returns (uint256) {
        return (collectionAdditionalData[_collectionID].collectionCirculationSupply - burnAmount[_collectionID]);
    }
```
[438-440](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L438-L440), [461-463](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L461-L463)

***Recommended Mitigation Steps***
Two options:
1. Explain the difference in documentation
2. If the struct variable `collectionTotalSupply` refers to the initial total supply (before any tokens are burned), then rename it to `collectionMaxSupply` (preferred option).

### L-05: `getPrice` would return zero if the collection does not exist or its data is not set yet
Recommendation: Check if the collection has data added before calculating the prices. For instance, add `require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data to a collection");` at the beginning of `getPrice` function: [530-568](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L530-L568)

## XRandoms.sol - informational

- Redundant function: `returnIndex()` returns the same result as `getWord` does. Set `getWord()` function visibility to `public` and remove `returnIndex()` function.
[XRandoms.sol#L15](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L15), [XRandoms.sol#L45-L47](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L45-L47)

## RandomizerRNG.sol - informational

- The `Withdraw` event only shows the initiator's address but not the receiver's (owner); this might be insufficient to understand for the event's reader. Add the receiver's address (msg.sender) as an event argument.
[RandomizerRNG.sol#L24](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L24), [RandomizerRNG.sol#L83](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L83)


## AuctionDemo.sol - informational

- Replace `address indexed _add` with `address indexed _address` to improve the code readability.
**There are 7 instances of this issue**
```solidity
File: smart-contracts/AuctionDemo.sol

22:             event ClaimAuction(address indexed _add, uint256 indexed tokenid, bool status, uint256 indexed funds);

23:             event Refund(address indexed _add, uint256 indexed tokenid, bool status, uint256 indexed funds);

24:             event CancelBid(address indexed _add, uint256 indexed tokenid, uint256 index, bool status, uint256 indexed funds);
```
[[22](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L22), [23](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L23), [24](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L24)]

```solidity
File: smart-contracts/MinterContract.sol

124:            event PayArtist(address indexed _add, bool status, uint256 indexed funds);

125:            event PayTeam(address indexed _add, bool status, uint256 indexed funds);

126:            event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```
[[124](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L124), [125](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L125), [126](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/MinterContract.sol#L126)]

```solidity
File: smart-contracts/RandomizerRNG.sol

24:             event Withdraw(address indexed _add, bool status, uint256 indexed funds);
```
[[24](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/RandomizerRNG.sol#L24)]

- Replace `auctionInfoStru` with `AuctionInfoStruct` to improve the code readability and follow the Solidity naming convention.

```solidity
File: smart-contracts/AuctionDemo.sol

43:             struct auctionInfoStru {
```
[[43](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L43)]

## NextGenAdmins.sol - informational

- Any function that registers admins can also remove them by just passing `_status` as `false`. The removal functionality is not included in documentation at the moment, which might confuse users who are not proficient in reading Solidity code. Instances:
[38](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L38), [44](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L44), [50](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L50), [58](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L58)

- If `NextGenAdmins` contract owner deregisters themselves, they will lose `GlobalAdmin` status, which might confuse the owner and users. Recommendation: in `registerAdmin()` add `require(msg.sender != owner()), "You are an owner!"` 
[38-40](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L38-L40)

## NextGenCore.sol - informational

- Unnecessary storage of both the randomizer address and its contract:
```solidity
    struct collectionAdditonalDataStructure {
        // ... other code
        uint256 setFinalSupplyTimeAfterMint;
        address randomizerContract;
    }
```
[52-53](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L52-L53)
Recommendation: 
    1. Remove the `randomizerContract` variable in the `collectionAdditonalDataStructure` struct
    [52](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L52)
    2. Remove the line [172](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L172)
    3. Replace `collectionAdditionalData[_collectionID].randomizerContract` with `address(collectionAdditionalData[_collectionID].randomizer)` in `setTokenHash` and `retrieveCollectionAdditionalData` functions [300](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L300), [438](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L438)

- Unnecessary mapping `artistSignatures`, as `artistSigned` would be enough. Remove the `artistSignatures` mapping and `_signature` parameter in the `artistSignature()` function to save gas and improve readability. Instances in NextGenCore.sol: [89](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L89), [260](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L260), [257](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L257)
Alternatively, `artistSigned` can be removed and in every check can be used `artistsSignatures[_collectionID] != ""`; however, this would also require an additional check for empty strings in the `artistSignature()` function.

- Artists can sign with empty strings. In order to prevent this, add a check for empty strings `require  (_signature != "")` in the `artistSignature()` function. [257-262](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L257-L262)

- The `NextGenCore.sol` constructor set calls `ERC2981._setDefaultRoyalty` setting 6.9% royalty rate to an unknown address `0x1B1289E34Fe05019511d7b436a5138F361904df0`, however, the usage of additional royalties on top of MinterContract collection royalties is not justified and might confuse users.
Recommendation: either explain the reason for NextGenCore royalties and its difference from MinterContract's royalties in documentation or remove `_setDefaultRoyalty(0x1B1289E34Fe05019511d7b436a5138F361904df0, 690);` from the constructor. [111](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L111)

## Multi-contract non-critical issues
- Make sure your contract name and .sol file name are the same for better readability, organization, compatibility, and consistency in the codebase. 
Rename the following contract names:
```
AuctionDemo.sol:    auctionDemo → AuctionDemo
MinterContract.sol: NextGenMinterContract → MinterContract
RandomizerNXT.sol:  NextGenRandomizerNXT → RandomizerNXT
RandomizerRNG.sol:  NextGenRandomizerRNG → RandomizerRNG
RandomizerVRF.sol:  NextGenRandomizerVRF → RandomizerVRF
XRandoms.sol:       randomPool → XRandoms
```

- INFO: Some supply checks and logic can be moved to a single function, `_mintProcessing` instead of having them in every function that leads to a token mint. 

In order to increase readability and make it easier to spot potential bugs and inconsistencies, it is recommended to remove the following instances:
```solidity
File: smart-contracts/NextGenCore.sol

180:    collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
181:    if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {

191:    collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
192:    if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {

216:    collectionAdditionalData[_mintCollectionID].collectionCirculationSupply = collectionAdditionalData[_mintCollectionID].collectionCirculationSupply + 1;
217:    if (collectionAdditionalData[_mintCollectionID].collectionTotalSupply >= collectionAdditionalData[_mintCollectionID].collectionCirculationSupply) {
```
[180-181](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L180-L181), [191-192](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L191-L192), [216-217](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L216-L217)

```solidity
File: smart-contracts/MinterContract.sol

183:    uint256 collectionTokenMintIndex;

185:    collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID) + _numberOfTokens[y] - 1;
186:    require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

230:    uint256 collectionTokenMintIndex;
231:    collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col) + _numberOfTokens - 1;
232:    require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");

263:    uint256 collectionTokenMintIndex;
264:    collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
265:    require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_mintCollectionID), "No supply");

278:    uint256 collectionTokenMintIndex;
279:    collectionTokenMintIndex = gencore.viewTokensIndexMin(_collectionID) + gencore.viewCirSupply(_collectionID);
280:    require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(_collectionID), "No supply");

358:        uint256 collectionTokenMintIndex;
359:        collectionTokenMintIndex = gencore.viewTokensIndexMin(col) + gencore.viewCirSupply(col);
360:        require(collectionTokenMintIndex <= gencore.viewTokensIndexMax(col), "No supply");
```
[183](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L183), [185-186](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L185-L186), [230-232](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L230-L232), [263-265](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L263-L265), [278-280](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L278-L280), [358-360](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L358-L360)

Instead, in the `_mintProcessing` functionality of NextGenCore.sol, add a single supply check, supply increase logic, and a mintIndex generator:
```solidity
    require(_mintIndex <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex, "No supply");
    ++collectionAdditionalData[_collectionID].collectionCirculationSupply;
```
Also, `mintIndex` can be generated at the last stages in `gencore._mintProcessing` instead of calculating it in MinterContract.sol function.



## Incorrect grammar
- Rename `participateToAuction` to `participateInAuction`

```solidity
File: smart-contracts/AuctionDemo.sol

57:             function participateToAuction(uint256 _tokenid) public payable {
```
[AuctionDemo.sol#L57](https://github.com/code-423n4/2023-10-nextgen/blob/08a56bacd286ee52433670f3bb73a0e4a4525dd4/smart-contracts/AuctionDemo.sol#L57)

- Rename `maxCollectionPurchases` to `maxMintsPerAddress` or `maxAllowance` in all instances for better readability. According to the documentation, the `maxCollectionPurchases` variable relates to the maximum amount of tokens allowed to be minted per address. Users or other developers can misunderstand the current naming:
```solidity
    struct collectionAdditonalDataStructure {
        // ... other code
        uint256 maxCollectionPurchases;
        // ... other code
    }
```
NextGenCore.sol: [46](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L46), [145](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L145), [147](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L147), [151](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L151), [160](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L160), [163](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L163), [400](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L400), [439](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L439)

- Rename the `wereDataAdded` variable to `isDataAdded` and `retrievewereDataAdded` to `retrieveIsDataAdded` in order to improve the readability of the code and make it more consistent with common naming conventions. Instances in NextGenCore.sol: 
[65](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L65), [157](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L157), [377](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L377), [378](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L378)

- Rename the `_recipient` parameter to `_mintTo` (as it is in the rest of the codebase) in `airDropTokens` and `_mintProcessing` in order to improve the readability of the code and make it more consistent with common naming conventions. Instances in NextGenCore.sol: [178](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L178), [182](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L182), [183](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L183), [227](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L227), [231](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L231)

- Rename the `freezeCollection` function to `permanentlyFreezeCollection` to provide developers and users with a clearer understanding of the long-term impact of invoking that function. NextGenCore.sol: [292](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L292)

- Rename the `viewColIDforTokenID` function to `viewCollectionIDforTokenID` to improve the readability of the code and make it more consistent with common naming conventions. NextGenCore.sol: [372](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L372)

- Rename the `retrieveTokensMintedALPerAddress` function to `retrieveAllowlistTokensMintedByAddress` to improve the readability of the code and make it more consistent with common naming conventions. NextGenCore.sol: [404](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L404)

- Rename the following mappings to improve the readability of the code in MinterContract.sol:
`collectionTotalAmount` to `collectionFundsRaised`: [23](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L23)
`mintToAuctionData` to `tokenToAuctionEndTime`: [112](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L112)
`mintToAuctionStatus` to `tokenToAuctionStatus`: [115](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L115)
`gencore` to `gencoreContract` for naming consistency: [118](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L118)

- Rename the following error messages to improve the readability and user experience in MinterContract.sol: 
 `AL limit` to `Reached Allowance Limit` in 2 instances: [213](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L213), [217](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L217)
 `Wrong ETH` to `Not Enough ETH` in 3 instances: [233](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L233), [266](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L266), [361](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L361)
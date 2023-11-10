## [L-1] NextGenCore does not make use of the ERC2981 standard

The ERC2981 standard specifies that the NFT contract that implements it should define a `royaltyInfo(uint256 tokenId, uint256 salePrice)` view function that returns 2 things:
1. The royalty fee amount for a token `tokenId` with a sale price `salePrice`
2. The address which is entitled to the royalty fees

The `NextGenCore` contract inherits OZ's ERC2981 implementation, but makes no use of it except that it sets the default royalty fee. It never sets the `receiver` of a royalty fee as the EIP suggests thus marketplaces on which NextGen core NFTs are traded on that support paying royalty fees to artists cannot help with paying royalty fees to artists. Plus, the NextGen protocol has no mechanism of distributing these royalty fees to artists.

## [NC-1] Declare all state variables at top level

Smart contract organization best practices advocate to put state variables at the very top of the contract.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L49-L53

```solidity

    // mapping of collectionSecondaryAddresses struct
    mapping (uint256 => auctionInfoStru[]) public auctionInfoData;

    // claim auctioned
    mapping (uint256 => bool) public auctionClaim;
```

## [NC-2] No need of access modifier on constructor

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L36

```solidity
    constructor(address _minter, address _gencore, address _adminsContract) {
```

## [NC-3] Declare modifiers after constructor

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L30-L34

```solidity
    // certain functions can only be called by auction winner or admin
    modifier WinnerOrAdminRequired(uint256 _tokenId, bytes4 _selector) {
      require(msg.sender == returnHighestBidder(_tokenId) || adminsContract.retrieveFunctionAdmin(msg.sender, _selector) == true || adminsContract.retrieveGlobalAdmin(msg.sender) == true, "Not allowed");
      _;
    }

    constructor (address _minter, address _gencore, address _adminsContract) public {
        minter = IMinterContract(_minter);
        gencore = _gencore;
        adminsContract = INextGenAdmins(_adminsContract);
    }
```

## [NC-4] Extract all collections delegation address occurrences to a constant

Extracting the `0x8888888888888888888888888888888888888888` address to a constant will improve code readability.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L333

```solidity
                isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(_delegator, 0x8888888888888888888888888888888888888888, msg.sender, 2);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L207
```solidity
            isAllowedToMint = dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 1) || dmc.retrieveGlobalStatusOfDelegation(ownerOfToken, 0x8888888888888888888888888888888888888888, msg.sender, 2);
```

## [NC-5] Not needed comparison of boolean values to `true`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L171
```solidity
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L158
```solidity
        require(gencore.retrievewereDataAdded(_collectionID) == true, "Add data");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L197
```solidity
        require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L317
```solidity
        require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L328
```solidity
        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L328
```solidity
        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L337
```solidity
            require(isAllowedToMint == true, "No delegation");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L171
```solidity
        require(IRandomizer(_randomizerContract).isRandomizerContract() == true, "Contract is not Randomizer");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L267
```solidity
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L293
```solidity
        require(isCollectionCreated[_collectionID] == true, "No Col");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L62
```solidity
        require(INextGenAdmins(_newadminsContract).isAdminContract() == true, "Contract is not Admin");
```


## [NC-5] Useless declaration outside of loop

Variable can be declared directly on [line 185](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L185)
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L183C1-L183C42

```
        uint256 collectionTokenMintIndex;
```

## [NC-6] Useless pair of brackets in expression

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L233
```solidity
        require(msg.value >= (getPrice(col) * _numberOfTokens), "Wrong ETH");
```
No need of brackets around `getPrice(col) * _numberOfTokens`.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L252
```solidity
            lastMintDate[col] = collectionPhases[col].allowlistStartTime + (collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1));
```
No need of brackets around `collectionPhases[col].timePeriod * (gencore.viewCirSupply(col) - 1)`.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536
```solidity
                return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));
```
No need of brackets around `collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate * gencore.viewCirSupply(_collectionId)`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L551
```solidity
                decreaserate = ((price - (collectionPhases[_collectionId].collectionMintCost / (tDiff + 2))) / collectionPhases[_collectionId].timePeriod) * ((block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime));
```
No need of brackets around `collectionPhases[_collectionId].collectionMintCost / (tDiff + 2)` and `(block.timestamp - (tDiff * collectionPhases[_collectionId].timePeriod) - collectionPhases[_collectionId].allowlistStartTime)`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L553
```solidity
                if (((collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate)) > tDiff) {
```
No need of brackets around `collectionPhases[_collectionId].collectionMintCost - collectionPhases[_collectionId].collectionEndMintCost) / (collectionPhases[_collectionId].rate`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L310
```solidity
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
```
No need of brackets around `_collectionID * 10000000000`.

```solidity
        require ((_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex) && (_tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex), "id err");
```
No need of brackets around `_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex` and `_tokenId >= collectionAdditionalData[_collectionID].reservedMinTokensIndex && _tokenId <= collectionAdditionalData[_collectionID].reservedMaxTokensIndex`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L267
```solidity
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false), "Not allowed");
```
No need of brackets around `isCollectionCreated[_collectionID] == true` and `collectionFreeze[_collectionID] == false` 

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361
```solidity
        require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
```
No need of brackets around `getPrice(col) * 1`

## [NC-7] Can define variable value only once

In both the `if` and the `else if` statements we define `mintingAddress = ownerOfToken;`. In the `else` statement we revert ⇒ can set `address mintingAddress` value directly at [line 342](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L342)

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L349
```solidity
            mintingAddress = ownerOfToken;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L353
```solidity
            mintingAddress = ownerOfToken;
```

## [NC-8] Unused function

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L488-L490

```solidity
    function retrieveSecondaryAddressesAndPercentages(uint256 _collectionID) public view returns(address, address, address, uint256, uint256, uint256, bool){
        return (collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1, collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2, collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3, collectionArtistSecondaryAddresses[_collectionID].add1Percentage, collectionArtistSecondaryAddresses[_collectionID].add2Percentage, collectionArtistSecondaryAddresses[_collectionID].add3Percentage, collectionArtistSecondaryAddresses[_collectionID].status);
    }
```

## [NC-9] Structs with identical attributes

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L86-L91
```solidity
    // royalties secondary splits structure

    struct royaltiesSecondarySplits {
        uint256 artistPercentage;
        uint256 teamPercentage;
    }
```
and
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L61-L66
```solidity
    // royalties primary splits structure

    struct royaltiesPrimarySplits {
        uint256 artistPercentage;
        uint256 teamPercentage;
    }
```

## [NC-10] Extract max supply value to a constant

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L148
```solidity
        require((isCollectionCreated[_collectionID] == true) && (collectionFreeze[_collectionID] == false) && (_collectionTotalSupply <= 10000000000), "err/freezed");
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L155
```solidity
            collectionAdditionalData[_collectionID].reservedMinTokensIndex = (_collectionID * 10000000000);
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L156
```solidity
            collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + _collectionTotalSupply - 1;
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L310
```solidity
        collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
```

## [NC-12] Typo in struct name

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L44
```solidity
    // collectionAdditionalData struct declaration
    struct collectionAdditonalDataStructure {
```
Should be `collectionAdditionalDataStructure`.

## [NC-13] Centralization risk

Admins can withdraw entire ETH balance.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L461-L466
```solidity
    function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        uint balance = address(this).balance;
        address admin = adminsContract.owner();
        (bool success, ) = payable(admin).call{value: balance}("");
        emit Withdraw(msg.sender, success, balance);
    }
```

## [NC-14] Change function visibility to `pure`

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L45-L47
```solidity
    function returnIndex(uint256 id) public view returns (string memory) {
        return getWord(id);
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L506-L508
```solidity
    function isMinterContract() external view returns (bool) {
        return true;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol#L83-L85
```solidity
    function isAdminContract() external view returns (bool) {
        return true;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L62-L64
```solidity
    function isRandomizerContract() external view returns (bool) {
        return true;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L105-L107
```solidity
    function isRandomizerContract() external view returns (bool) {
        return true;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L89-L91
```solidity
    function isRandomizerContract() external view returns (bool) {
        return true;
    }
```
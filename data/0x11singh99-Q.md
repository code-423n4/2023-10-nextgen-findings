# Low-Level and Non-Critical issues

## Summary

### Low Level List

| Number | Issue Details                                                                                                                   |     |
| :----: | :------------------------------------------------------------------------------------------------------------------------------ | :-: |
| [L-01] | Avoid using `raw percentage` just 1 to 100 number                                                                               |
| [L-02] | Use `pull` over `push` to transfer ether to artists to avoid DOS                                                                |
| [L-03] | Loss of precision                                                                                                               |
| [L-04] | Use `nonReentrant` modifier of Openzeppelin's `ReentrancyGuard` in the functions handling transfers of assets especially ethers |

Total 3 Low Level Issues

### Non Critical List

| Number  | Issue Details                                                      |
| :-----: | :----------------------------------------------------------------- |
| [NC-01] | Use uint256 instead of uint for consistency                        |
| [NC-02] | Typo                                                               |
| [NC-03] | Confusing comments                                                 |
| [NC-04] | Use proper Nat Spec                                                |
| [NC-05] | Would be better if function name was `setAdditionalCollectionData` |
| [NC-06] | Using Openzeppelin's counters library will be better choice        |
| [NC-07] | Missing `event` emission when changing state                       |

Total 4 Non Critical Issues

# LOW FINDINGS

## [L-01] Avoid using `raw percentage` just 1 to 100

Because solidity does not support floating numbers so use some extra zeros for decimals or BIPS to use exact percentage like 33.33% otherwise by using just 1 to 100 integers you won't able to use percentage like 33.33% or 50.5% etc.

`require(_artistPrSplit + _teamPrSplit == 100, "splits need to be 100%");`

As you can see in above require statement the number directly denoting `100 for 100%` use at least `10000 to denote 100% `

```solidity
File : smart-contracts/MinterContract.sol

63: struct royaltiesPrimarySplits {
64:      uint256 artistPercentage;
65:      uint256 teamPercentage;
66:  }

```

[63-66](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L63C5-L66C6)

## [L-02] Use `pull` over `push` to transfer ether to artists to avoid DOS

Because one artist can revert when he receives ether because of it all transfers will be halted to other artists

```solidity
File : smart-contracts/MinterContract.sol

434: (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties ("");
435: (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties ("");
436: (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties ("");
437: (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
438: (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");

```

[434-438](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L434C9-L438C74)

## [L-03] Loss of precision

Division before multiplication cause precision loss

```solidity
File : smart-contracts/MinterContract.sol

536:  return collectionPhases[_collectionId].collectionMintCost + ((collectionPhases[_collectionId].collectionMintCost / collectionPhases[_collectionId].rate) * gencore.viewCirSupply(_collectionId));

```

[536](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L536)

## [L-04] Use `nonReentrant` modifier of Openzeppelin's `ReentrancyGuard` in the functions handling transfers of assets especially ethers

To avoid any re-entrancy chance we must use non-reentrant modifier along with following CEI pattern

```solidity
File : smart-contracts/AuctionDemo.sol

124:    function cancelBid(uint256 _tokenid, uint256 index) public {
       ...
128:        (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");
        ...
    }


134: function cancelAllBids(uint256 _tokenid) public {
        ...
139:  (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
         ...
    }

```

[124-130](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L124C3-L130C6), [134-143](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L134C5-L143C6)

```solidity

415: function payArtist(uint256 _collectionID, address _team1, address _team2, uint256 _teamperc1, uint256 _teamperc2) public FunctionAdminRequired(this.payArtist.selector) {
    ...
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
      ...
444:   }


461: function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
        ...
        (bool success, ) = payable(admin).call{value: balance}("");
        ...
466:    }

```

[412-444](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L415C5-L444C6), [461-466](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L461C5-L466C6)

# NON-CRITICAL FINDINGS

## [NC-01] Use uint256 instead of uint for consistency

It is recommended to always use uint256/int256 instead of uint/int

**Note: missed by bot**

```solidity
File : smart-contracts/MinterContract.sol

26: mapping (uint256 => uint) public lastMintDate;

```

[26](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L26)

## [NC-02] Typo

```solidity

///@audit Additonal  should be Additional

44: struct collectionAdditonalDataStructure {

```

## [NC-03] Confusing comments

```solidity
File : smart-contracts/NextGenCore.sol

///@audit comments saying only two property will set if total supply but below finalSupplyTime is set

144: // once a collection is created and total supply is set it cannot be changed
145: // only _collectionArtistAddress , _maxCollectionPurchases can change after total supply is set

161: collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint = _setFinalSupplyTimeAfterMint;

```

[144-145](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L144C4-L145C100), [161](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L161)

## [NC-04] Use proper Nat Spec

```solidity
File : smart-contracts/NextGenCore.sol


178: function airDropTokens(uint256 mintIndex, address _recipient, string memory _tokenData, uint256 _saltfun_o, uint256 _collectionID) external {

```

[178](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L178)

## [NC-05] Would be better if function name was `setAdditionalCollectionData`

```solidity
File : smart-contracts/NextGenCore.sol

147: function setCollectionData(uint256 _collectionID, address _collectionArtistAddress, uint256 _maxCollectionPurchases, uint256 _collectionTotalSupply, uint _setFinalSupplyTimeAfterMint) public CollectionAdminRequired(_collectionID, this.setCollectionData.selector) {

```

[147](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147)

## [NC-06] Using Openzeppelin's `Counters` library will be better choice

When increasing one by one a Id then to maintain consistency and reduce scope of error. Use openzeppelin Counters library.

```solidity
File : smart-contracts/NextGenCore.sol

140: newCollectionIndex = newCollectionIndex + 1;

```

[140](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L140)

## [NC-07] Missing `event` emission when changing state

```solidity
File : smart-contracts/NextGenCore.sol

140: newCollectionIndex = newCollectionIndex + 1;//@audit emit events when changing imp. storage variable

```

[140](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L140)

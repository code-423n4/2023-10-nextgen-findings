# [L-01] Rounding errors can cause small artist payment loss

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L415

In `MinterContract.payArtist`, the calculation for payments to artist and team addresses may not add up to the total amount of royalties even though `collectionTotalAmount` is always set to zero. This results in small amounts that should have been paid to artists to stay in the contract.

These dust amounts can be withdrawn via `emergencyWithdraw`. The amounts are also limited to a very small 5 wei for each call to `payArtist`. Nevertheless, this could be mitigated by keeping this small amount in `collectionTotalAmount` to be collected by the artists in the future.

Current:
```solidity
uint256 royalties = collectionTotalAmount[_collectionID];
collectionTotalAmount[_collectionID] = 0;
address tm1 = _team1;
address tm2 = _team2;
uint256 colId = _collectionID;
uint256 artistRoyalties1;
uint256 artistRoyalties2;
uint256 artistRoyalties3;
uint256 teamRoyalties1;
uint256 teamRoyalties2;
artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
teamRoyalties1 = royalties * _teamperc1 / 100;
teamRoyalties2 = royalties * _teamperc2 / 100;
(bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
(bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
(bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
(bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
(bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```

Recommendation:
```solidity
uint256 royalties = collectionTotalAmount[_collectionID];
collectionTotalAmount[_collectionID] = 0;
address tm1 = _team1;
address tm2 = _team2;
uint256 colId = _collectionID;
uint256 artistRoyalties1;
uint256 artistRoyalties2;
uint256 artistRoyalties3;
uint256 teamRoyalties1;
uint256 teamRoyalties2;
artistRoyalties1 = royalties * collectionArtistPrimaryAddresses[colId].add1Percentage / 100;
artistRoyalties2 = royalties * collectionArtistPrimaryAddresses[colId].add2Percentage / 100;
artistRoyalties3 = royalties * collectionArtistPrimaryAddresses[colId].add3Percentage / 100;
teamRoyalties1 = royalties * _teamperc1 / 100;
teamRoyalties2 = royalties * _teamperc2 / 100;

(bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
(bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
(bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
(bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
(bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");

// NEW, after transfers to avoid reentrancy
collectionTotalAmount[_collectionID] = royalties - artistRoyalties1 -artistRoyalties2 - artistRoyalties3 - teamRoyalties1 - teamRoyalties2;
```

# [NC-01] `reservedMaxTokensIndex` can be less than `reservedMinTokensIndex`

In `NextGenCore.sol`, the maximum index for a token can be 1 less than the minimum index if the supply is zero. No issues were found that can be cause by this, but the incorrect semantics could cause unforeseen problems with anyone reading these values offchain.

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L155-L156

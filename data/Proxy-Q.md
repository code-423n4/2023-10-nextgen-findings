# Low and Non-Critical

### [Low Risk](#low-risk-1)

| Total Low Risk Issues | 2 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [L-01](#l-01-minting-phases-can-be-set-to-overlap) | Minting phases can be set to overlap | 1 |
| [L-02](#l-02-_mintprocessing-should-not-be-able-to-be-called-if-totalsupply--circulatingsupply) | `_mintProcessing()` should not be able to be called if `totalSupply == circulatingSupply` | 3 |

### [Non-Critical](#non-critical-1)

| Total Non-Critical Issues | 2 |
|:--:|:--:|

| Count | Title | Instances |
|:--:|:-------| :--: |
| [NC-01](#nc-01-combine-returnhighestbid-and-returnhighestbidder-into-one-function) | Combine `returnHighestBid` and `returnHighestBidder` into one function | 1 |
| [NC-02](#nc-02-addmintercontract-should-be-called-in-constructor) | `addMinterContract()` should be called in constructor | 1 |

## Low Risk

### [L-01] Minting phases can be set to overlap

Allowlist minting phases and Public minting phases should not overlap. Add `require` statements to prevent this. 

```solidity
function setCollectionPhases(
    uint256 _collectionID,
    uint256 _allowlistStartTime,
    uint256 _allowlistEndTime,
    uint256 _publicStartTime,
    uint256 _publicEndTime,
    bytes32 _merkleRoot
) public CollectionAdminRequired(_collectionID, this.setCollectionPhases.selector) {
    require(setMintingCosts[_collectionID] == true, "Set Minting Costs");
    collectionPhases[_collectionID].allowlistStartTime = _allowlistStartTime; // @audit-issue L: Minting phases can be set to overlap
    collectionPhases[_collectionID].allowlistEndTime = _allowlistEndTime;
    collectionPhases[_collectionID].merkleRoot = _merkleRoot;
    collectionPhases[_collectionID].publicStartTime = _publicStartTime;
    collectionPhases[_collectionID].publicEndTime = _publicEndTime;
}
```

### [L-02] `_mintProcessing()` should not be able to be called if `totalSupply == circulatingSupply`

In `airDropTokens()`, `mint()` and `burnToMint()`. Change from `>=` to `>`.

[airDropTokens](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L181-L182)

[mint](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L192-L193)

[burnToMint](https://github.com/code-423n4/2023-10-nextgen/blob/main/hardhat/smart-contracts/NextGenCore.sol#L217-L218)

```solidity
if (
    collectionAdditionalData[_collectionID].collectionTotalSupply
        >= collectionAdditionalData[_collectionID].collectionCirculationSupply
) {
    _mintProcessing(mintIndex, _recipient, _tokenData, _collectionID, _saltfun_o);
    ...
```

## Non-critical

### [NC-01] Combine `returnHighestBid` and `returnHighestBidder` into one function

`returnHighestBid()` and `returnHighestBidder()` have the same logic. Combine them together and return the highest bidder and the highest bid. It reduces gas costs also.

### [NC-02] `addMinterContract()` should be called in constructor

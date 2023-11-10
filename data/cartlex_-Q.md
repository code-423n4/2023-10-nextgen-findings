| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-setfinalsupply-function-can-be-used-before-collection-is-created) | `setFinalSupply()` function can be used before collection is created | Low |
| [L-02](#l-02-incorrect-implementation-of-emergencywithdraw)| Incorrect Implementation of `emergencyWithdraw()` | Low |
| [NC-01](#nc-01-keyhash-parameter-used-from-goerli-instead-of-mainnet)| `keyHash` parameter used from goerli instead of mainnet | Non-critical |
| [NC-02](#nc-02-proposesecondaryaddressesandpercentages-function-is-not-used-anywhere)| `proposeSecondaryAddressesAndPercentages()` function is not used anywhere | Non-critical |

# [L-01] `setFinalSupply()` function can be used before collection is created

It's possible to call `setFinalSupply()` function with collection id that is no exist.

```solidity
function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
    require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
    collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
    collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
}
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L307-L311

## Recommended Mitigation Steps
Consider to implement additional check that it's not possible to call `setFinalSupply()` function when collection is not created.

```diff
function setFinalSupply(uint256 _collectionID) public FunctionAdminRequired(this.setFinalSupply.selector) {
++  require(isCollectionCreated[_collectionID] == true, "Collection not created");
    require (block.timestamp > IMinterContract(minterContract).getEndTime(_collectionID) + collectionAdditionalData[_collectionID].setFinalSupplyTimeAfterMint, "Time has not passed");
    collectionAdditionalData[_collectionID].collectionTotalSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply;
    collectionAdditionalData[_collectionID].reservedMaxTokensIndex = (_collectionID * 10000000000) + collectionAdditionalData[_collectionID].collectionTotalSupply - 1;
    }
```


# [L-02] Incorrect Implementation of `emergencyWithdraw()`

There is an external call that violate the principles of an emergency withdrawal. This call introduce points of failure, which if influenced by the contract's owner or due to errors in the implementations, could cause the 'emergencyWithdraw' function to revert, compromising its primary feature.

```solidity
function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
    uint balance = address(this).balance;
    address admin = adminsContract.owner();
    (bool success, ) = payable(admin).call{value: balance}("");
    emit Withdraw(msg.sender, success, balance);
}
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L461-L466

## Recommended Mitigation Steps
Modify the implementation to eliminate unnecessary restrictions and calls. For example, refer to this [Pancakeswap Smart Contract](https://github.com/pancakeswap/pancake-smart-contracts/blob/d8f55093a43a7e8913f7730cfff3589a46f5c014/projects/smartchef/v2/contracts/SmartChefInitializable.sol#L235-L246) for a better approach.


# [NC-01] `keyHash` parameter used from goerli instead of mainnet

NextGen protocol will be deployed on `mainnet` due to information in `README.md` file, however, in `RandomizerVRF.sol` contract `keyHash` variable use Chainlink VRF hash from `goerli` network.

```solidity
bytes32 public keyHash = 0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15;
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L26

## Recommended Mitigation Steps
Consider to change `keyHash` from `goerli` to `mainnet`.


# [NC-02] `proposeSecondaryAddressesAndPercentages()` function is not used anywhere

In `MinterContract.sol` there is a function `proposeSecondaryAddressesAndPercentages()`.
This function is not used anywhere.

```solidity
function proposeSecondaryAddressesAndPercentages(uint256 _collectionID, address _secondaryAdd1, address _secondaryAdd2, address _secondaryAdd3, uint256 _add1Percentage, uint256 _add2Percentage, uint256 _add3Percentage) public ArtistOrAdminRequired(_collectionID, this.proposeSecondaryAddressesAndPercentages.selector) {
        require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
        require (_add1Percentage + _add2Percentage + _add3Percentage == collectionRoyaltiesSecondarySplits[_collectionID].artistPercentage, "Check %");
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd1 = _secondaryAdd1;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd2 = _secondaryAdd2;
        collectionArtistSecondaryAddresses[_collectionID].secondaryAdd3 = _secondaryAdd3;
        collectionArtistSecondaryAddresses[_collectionID].add1Percentage = _add1Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add2Percentage = _add2Percentage;
        collectionArtistSecondaryAddresses[_collectionID].add3Percentage = _add3Percentage;
        collectionArtistSecondaryAddresses[_collectionID].status = false;
    }
```

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L394-L404

## Recommended Mitigation Steps
Consider to remove this function or implement additional logic for it.
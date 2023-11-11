**[[1]]** Function `NextGenMinterContract.mint` contains a useless check:
```solidity
// This condition always holds
require(_numberOfTokens <= gencore.viewMaxAllowance(col), "Change no of tokens");
// If this condition holds
require(gencore.retrieveTokensMintedPublicPerAddress(col, msg.sender) + _numberOfTokens <= gencore.viewMaxAllowance(col), "Max");
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L223

**[[2]]** Function `NextGenMinterContract.burnToMint` contains a useless variable:
```solidity
...
uint256 collectionTokenMintIndex;
collectionTokenMintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
...
// It has exactly the same value as the collectionTokenMintIndex
uint256 mintIndex = gencore.viewTokensIndexMin(_mintCollectionID) + gencore.viewCirSupply(_mintCollectionID);
```
There are similar cases in the `NextGenMinterContract.mintAndAuction`, `NextGenMinterContract.burnOrSwapExternalToMint`.
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L267
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L281
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L362

**[[3]]** The statement `collectionArtistPrimaryAddresses[_collectionID].status = false` in the `NextGenMinterContract.proposePrimaryAddressesAndPercentages` and the `NextGenMinterContract.proposeSecondaryAddressesAndPercentages` is useless since it is guaranteed to be false.
```solidity
require (collectionArtistSecondaryAddresses[_collectionID].status == false, "Already approved");
...
collectionArtistSecondaryAddresses[_collectionID].status = false;
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L389
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L403

**[[4]]** Function `fulfillRandomWords` in the `NextGenRandomizerRNG` and `NextGenRandomizerVRF` contains the construction:
```solidity
bytes32(abi.encodePacked(numbers,requestToToken[id]))
```
This construction is equivalent to:
```solidity
bytes32(numbers[0])
```
So it is not clear why this construction is used. I guess it was assumed to be:
```solidity
bytes32(kecak256(abi.encodePacked(numbers,requestToToken[id])))
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerRNG.sol#L49
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerVRF.sol#L66

**[[5]]** The word "Watermelon" is never returned in the `randomPool.randomWord`. The max value of the `randomNum` is 99. In this case, `getWord` will return `wordsList[99 - 1]` that is "Velvet Apple".

https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/XRandoms.sol#L40

**[[6]]** Function `NextGenRandomizerNXT.calculateTokenHash` looks a bit overcomplicated. In terms of randomness it should be equivalent to:
```solidity
bytes32 hash = keccak256(abi.encodePacked(_mintIndex, block.prevrandao, blockhash(block.number - 1), block.timestamp);
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/RandomizerNXT.sol#L57

**[[7]]** There are multiple functions in the `NextGenCore` (`airDropTokens`, `mint`, `burnToMint`) with similar logic:
```solidity
require(msg.sender == minterContract, "Caller is not the Minter Contract");
collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;

if (collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply) {
    _mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
    if (phase == 1) {
        tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
    } else {
        tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
    }
}
```
If the `collectionCirculationSupply` is equal to `collectionTotalSupply - 1` then the `collectionCirculationSupply` will be increased but nothing will be minted. It is impossible now since this check is present in the minter contract. Still, it looks a bit fragile so it seems that it is better to replace `if` statement with `require` statement:
```solidity
require(msg.sender == minterContract, "Caller is not the Minter Contract");
collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;
require(collectionAdditionalData[_collectionID].collectionTotalSupply >= collectionAdditionalData[_collectionID].collectionCirculationSupply, "");

_mintProcessing(mintIndex, _mintTo, _tokenData, _collectionID, _saltfun_o);
if (phase == 1) {
    tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;
} else {
    tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;
}
```
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L181
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L192
https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L217

**[[8]]** There are a lot of places in the `NextGenCore` contract where the value `10000000000` is used. Consider introducing a new constant instead.
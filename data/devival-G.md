- GAS: ++i costs less gas compared to i = i + 1. To save gas, edit the following instances:

1. Replace `newCollectionIndex = newCollectionIndex + 1;` 
with `++newCollectionIndex;` 
in [NextGenCore.sol#L110](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L110), [NextGenCore.sol#L140](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L140).
2. Replace `collectionAdditionalData[_collectionID].collectionCirculationSupply = collectionAdditionalData[_collectionID].collectionCirculationSupply + 1;` 
with `++collectionAdditionalData[_collectionID].collectionCirculationSupply;` 
in [NextGenCore.sol#L180](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L180), [NextGenCore.sol#L191](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L191).
3. Replace `tokensMintedAllowlistAddress[_collectionID][_mintingAddress] = tokensMintedAllowlistAddress[_collectionID][_mintingAddress] + 1;` 
with `++tokensMintedAllowlistAddress[_collectionID][_mintingAddress];` 
in [NextGenCore.sol#L195](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L195).
4. Replace `tokensMintedPerAddress[_collectionID][_mintingAddress] = tokensMintedPerAddress[_collectionID][_mintingAddress] + 1;` 
with `++tokensMintedPerAddress[_collectionID][_mintingAddress];` 
in [NextGenCore.sol#L197](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L197).
5. Replace `burnAmount[_collectionID] = burnAmount[_collectionID] + 1;` 
with `++burnAmount[_collectionID];` 
in [NextGenCore.sol#L208](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L208).
6. Replace `burnAmount[_burnCollectionID] = burnAmount[_burnCollectionID] + 1;` 
with `++burnAmount[_burnCollectionID];` 
in [NextGenCore.sol#L221](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenCore.sol#L221).

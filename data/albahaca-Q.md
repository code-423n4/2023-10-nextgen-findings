## [L-01] Use of abi.encodePacked with dynamic types inside keccak256
abi.encodePacked should not be used with dynamic types when passing the result to a hash function such as keccak256. Use abi.encode instead, which will pad items to 32 bytes, to [prevent any hash collisions](https://docs.soliditylang.org/en/latest/abi-spec.html#non-standard-packed-mode).

note : these issue are missed from bots
```solidity
File: smart-contracts/MinterContract.sol
212  node = keccak256(abi.encodePacked(_delegator, _maxAllowance, tokData));

316  bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));

327  bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L212

## [L‑02] Use Ownable2Step instead of Ownable
Ownable2Step and Ownable2StepUpgradeable prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.


```solidity
File: smart-contracts/AuctionDemo.sol
18  contract auctionDemo is Ownable {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L18


```solidity
File: smart-contracts/MinterContract.sol
20  contract NextGenMinterContract is Ownable {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L20


```solidity
File: smart-contracts/RandomizerRNG.sol
18   contract NextGenRandomizerRNG is ArrngConsumer, Ownable {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L18


```solidity
File: smart-contracts/RandomizerVRF.sol
19   contract NextGenRandomizerVRF is VRFConsumerBaseV2, Ownable {
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L19


## [L-03] no check if the burn amount is zero or not
if the amount is zero so the unhecked block will divided zero on 3 and we use gas for nothing ! if we set zero we may pass the _burn checks, i know it is passed by only permit but it's better to avoid this happen because it's seting by human and it means it can be set with 0 balance to burn !



```solidity
File: smart-contracts/NextGenCore.sol
220   _burn(_tokenId);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L220

## [L-04] Gas griefing/theft is possible on unsafe external call
return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, 'out' and 'outsize' values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```solidity
File: smart-contracts/AuctionDemo.sol
113  (bool success, ) = payable(owner()).call{value: highestBid}("");

116  (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");

128  (bool success, ) = payable(auctionInfoData[_tokenid][index].bidder).call{value: auctionInfoData[_tokenid][index].bid}("");

139  (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L113

```solidity
File: smart-contracts/MinterContract.sol
        (bool success1, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd1).call{value: artistRoyalties1}("");
        (bool success2, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd2).call{value: artistRoyalties2}("");
        (bool success3, ) = payable(collectionArtistPrimaryAddresses[colId].primaryAdd3).call{value: artistRoyalties3}("");
        (bool success4, ) = payable(tm1).call{value: teamRoyalties1}("");
        (bool success5, ) = payable(tm2).call{value: teamRoyalties2}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L434-L438

```solidity
File: smart-contracts/RandomizerRNG.sol
82  (bool success, ) = payable(admin).call{value: balance}("");
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L82

## [L-05] Use fixed compiler version
All in scope contracts use ^0.8.19 as compiler version.


## [L-06] Missing Event for initialize
Description: Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip
Recommendation: Add Event-Emit



https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L157-L166

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L369-L376

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L170-L177

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L147-L166

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L307-L311

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L329-331

https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L299-L303


## [L-07] Incorrect from Parameter in IERC721.safeTransferFrom() in AuctionDemo Claim Auction Function

Description:
In the AuctionDemo.sol smart contract, specifically at line 112, the safeTransferFrom() function from the IERC721 interface is not being called correctly. The from parameter in the safeTransferFrom() function should be related to the msg.sender, but it appears to be using a different value.

```solidity
File: smart-contracts/AuctionDemo.sol
112   IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112

Recommendation:

from parameter must be related to msg.sender

## [L-08] Weak PRNG
Weak PRNG due to a modulo on `block.timestamp`, `now` or `blockhash`. These can be influenced by miners to some extent so they should be avoided.

```solidity
File: smart-contracts/XRandoms.sol
36   uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;

41   uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 100;
```
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/XRandoms.sol#L36

`Recommendation`:

Do not use `block.timestamp`, `now` or `blockhash` as a source of randomness
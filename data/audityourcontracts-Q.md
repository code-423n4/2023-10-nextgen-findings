## [01] Expired delegations can be used to mint tokens
`mint()` and `burnOrSwapExternalToMint()` check a delegation registry to allow a delegatee (the one receiving the delegation) to mint on behalf of a delegator (the one making the delegation).

If `mint()` is called with a delegator the MinderContract.sol will check the delegation using `retrieveGlobalStatusOfDelegation()`. It will check the delegation for all tokens `0x8888888888888888888888888888888888888888` and then more specifically for the `delAddress` of the collection. However the method it uses to check the delegation does not check whether the delegation has expired or not. If there is a past delegation and it has expired (but has not been removed) then `isAllowedToMint()` is set to true and the delegatee can mint on behalf of the delegator.

Note: The NFTDelegation.sol is out of scope but MinterContract.sol is in-scope and this is not a bug in the delegation system but rather how MinterContract.sol uses the delegation system. This is not a bug being raised within NFTDelegation.sol. There are other APIs that MinterContract.sol can use to make this safer.

The reason this is low is currently the only account the token can be minted to is the `_delegator` i.e. `mintingAddress = _delegator;`. However if this was changed to `msg.sender` in the future then the lack of expiry checking could be used to steal token mints.

## Recommendation

Use a delegation api that evaluates the expiry of the delegation such as `retrieveActiveDelegations()` which is demonstrated below;

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Test, console2} from "forge-std/Test.sol";
import "./smart-contracts/NextgenCore.sol";
import "./smart-contracts/NextgenAdmins.sol";
import "./smart-contracts/RandomizerRNG.sol";
import "./smart-contracts/MinterContract.sol";
import "./smart-contracts/NFTdelegation.sol";

contract DelegationTest is Test {
    NextGenAdmins admins;
    NextGenCore core;
    NextGenRandomizerRNG _randomizerContract;
    NextGenMinterContract minter;
    DelegationManagementContract registry;

    function setUp() public {
        admins = new NextGenAdmins();
        core = new NextGenCore("Nextgen", "NEXT", address(admins));
        registry = new DelegationManagementContract();
    }

    function test_Delegation() public {
       address _collectionAddress = address(core);
       address _delegationAddress = address(1337);
       uint256 _expiryDate = block.timestamp + 100;
       uint256 _useCase = 1;
       bool _allTokens = true;
       uint256 _tokenId = 0;
       registry.registerDelegationAddress(_collectionAddress, _delegationAddress, _expiryDate, _useCase, _allTokens, _tokenId);

       address[] memory activeDelegations = registry.retrieveActiveDelegations(address(this), _collectionAddress, block.timestamp, 1);
       vm.warp(block.timestamp + 1_000);
       bool isAllowed;
       address delegator = address(this);

       isAllowed = (registry.retrieveGlobalStatusOfDelegation(delegator, _collectionAddress, _delegationAddress, 1) || registry.retrieveGlobalStatusOfDelegation(delegator, _collectionAddress, _delegationAddress, 2));
       assert(true);

       address[] memory activeDelegationsExpired = registry.retrieveActiveDelegations(address(this), _collectionAddress, block.timestamp, 1);
    }
}
```

## [02] Auction token owner needs to explicitly approve AuctionDemo.sol contract
[L112](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L112) of AuctionDemo.sol. There's no note of explicit approval being required as there is in other areas of the code. 

## Recommendation
Comment code with regards to your assumptions. The winner never gets their token if this approval is not done. I am assuming that recipient of the auction token is trusted not to take off with the token and will explicitly approve and this will take place before the auction is claimed. That's why I haven't raised as a medium.

## [03] Empty else clause
[L118](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/AuctionDemo.sol#L118) of AuctionDemo.sol

```solidity
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
            } else {}
```

## Recommendation

Remove the empty else statement.

```diff
            if (auctionInfoData[_tokenid][i].bidder == highestBidder && auctionInfoData[_tokenid][i].bid == highestBid && auctionInfoData[_tokenid][i].status == true) {
                IERC721(gencore).safeTransferFrom(ownerOfToken, highestBidder, _tokenid);
                (bool success, ) = payable(owner()).call{value: highestBid}("");
                emit ClaimAuction(owner(), _tokenid, success, highestBid);
            } else if (auctionInfoData[_tokenid][i].status == true) {
                (bool success, ) = payable(auctionInfoData[_tokenid][i].bidder).call{value: auctionInfoData[_tokenid][i].bid}("");
                emit Refund(auctionInfoData[_tokenid][i].bidder, _tokenid, success, highestBid);
-           } else {}
+           } 
```

## [04] Validation should be performed in `setCollectionPhases()`
If the `_allowlistStartTime` and `_allowlistEndTime` are passed to `setCollectionPhases()` then `_merkleRoot` cannot be zero bytes. Otherwise all allowlist minting will fail with in invalid proof.

## Recommendation
`require` that `_merkleRoot` is not empty if allowlist times are populated.

## [05] `burnOrSwapExternalToMint()` doesn't have a `_burnCollectionID` within NextGenCore.sol
For external collection burns there's no associated burn collection in NextGenCore.sol.

## Recommendation
`_burnCollectionID` can be passed as zero but this can also cause confusion. It might be better for external collection burns to just use the `_mintCollectionID` for externalCol keccak256 hashes. For example;

```diff
-    function initializeExternalBurnOrSwap(address _erc721Collection, uint256 _burnCollectionID, uint256 _mintCollectionID, uint256 _tokmin, uint256 _tokmax, address _burnOrSwapAddress, bool _status) public FunctionAdminRequired(this.initializeExternalBurnOrSwap.selector) { 
+    function initializeExternalBurnOrSwap(address _erc721Collection, uint256 _mintCollectionID, uint256 _tokmin, uint256 _tokmax, address _burnOrSwapAddress, bool _status) public FunctionAdminRequired(this.initializeExternalBurnOrSwap.selector) { 
-        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
+        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_mintCollectionID));
        require((gencore.retrievewereDataAdded(_mintCollectionID) == true), "No data");
        burnExternalToMintCollections[externalCol][_mintCollectionID] = _status;
        burnOrSwapAddress[externalCol] = _burnOrSwapAddress;
        burnOrSwapIds[externalCol][0] = _tokmin;
        burnOrSwapIds[externalCol][1] = _tokmax;
    }
```

And 

```diff
-    function burnOrSwapExternalToMint(address _erc721Collection, uint256 _burnCollectionID, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
+    function burnOrSwapExternalToMint(address _erc721Collection, uint256 _tokenId, uint256 _mintCollectionID, string memory _tokenData, bytes32[] calldata merkleProof, uint256 _saltfun_o) public payable {
-        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_burnCollectionID));
+        bytes32 externalCol = keccak256(abi.encodePacked(_erc721Collection,_mintCollectionID));
        require(burnExternalToMintCollections[externalCol][_mintCollectionID] == true, "Initialize external burn");
        require(setMintingCosts[_mintCollectionID] == true, "Set Minting Costs");
        address ownerOfToken = IERC721(_erc721Collection).ownerOf(_tokenId);

```

## [06] Multiplying by single token not required 
[L361](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L361) of MinterContract.sol multiplies by a single token.

## Recommendation

This can be removed;

```diff
-       require(msg.value >= (getPrice(col) * 1), "Wrong ETH");
+       require(msg.value >= getPrice(col), "Wrong ETH");
```

## [07] `_saltfun_o` is never used randomizers
I understand it's a future setting but it's never used in a number of functions. Especially the randomizers.

[L53](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L53) of RandomizerRNG.sol
[L71](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L71) of RandomizerVRF.sol
[L55](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L55) of RandomizerNXT.sol

## Recommendation
These references should be removed.

## [08] `fulfillRandomWords()` in RandomizerRNG.sol should emit RequstFulfilled event
Events are not consistently emitted across randomizers. VRF [emits](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerVRF.sol#L67) an event when `fulfillRandomWords()` but RandomizerRNG does not see [L48](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerRNG.sol#L48).

## Recommendation
Add the event to RandomizerRNG.sol and emit it similar to RandomizerVRF.sol.
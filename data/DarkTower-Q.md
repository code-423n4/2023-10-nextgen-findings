## Summary

### Low Risk Issues
|Tag |Issue|Contexts|
|-|:-|:-:|
|LOW-1| Function Admin centralization risk in `emergencyWithdraw` function |1| 
|LOW-2| Bad Source of Randomness is implemented in `NextGenRandomizerNXT::calculateTokenHash`|1|
|LOW-3| Improve logic in function `auctionDemo::returnHighestBidder`

Total: 2 contexts over 2 issues

### LOW-1: Function Admin centralization risk in `emergencyWithdraw` function
A malicious function admin can reset the `MinterContract` balance to zero anytime they choose thereby disrupting usual accounting an flow of funds. Keep in mind since this contract holds the funds from minting for all collections, an admin calling it everytime will end up wiring the contract's balance to the owner but that will cause tedious work when reimbursing each individual artist of a collection for example as the collection could easily be over a hundred to thousands of collections' funds wired now needing reimbursement and temporarily pausing crucial functions such as `payArtist` if the contract has no balance.

Context: [MinterContract::emergencyWithdraw](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L461)
```solidity
        function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
              uint balance = address(this).balance;
              address admin = adminsContract.owner();
              (bool success, ) = payable(admin).call{value: balance}("");
              emit Withdraw(msg.sender, success, balance);
    }
```
LOW-2: Bad Source of Randomness is implemented in `NextGenRandomizerNXT::calculateTokenHash`
Using block.number and block.timestamp as a source of randomness is commonly advised against, as the outcome can be manipulated by calling contracts. In this case, generating unique hash for tokens IDs can be exploited during minting process.

Context: [NextGenRandomizerNXT::calculateTokenHash](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/RandomizerNXT.sol#L57)
```
function randomNumber() public view returns (uint256){
@>      uint256 randomNum = uint(keccak256(abi.encodePacked(block.prevrandao, blockhash(block.number - 1), block.timestamp))) % 1000;
        return randomNum;
    }
```

LOW-3: Improve logic in function `auctionDemo::returnHighestBidder`
As of now the `highBid` is not getting updated and the function always compares `auctionInfoData[_tokenid][i].bid` to 0. This works for now as the mapping `mapping (uint256 => auctionInfoStru[]) public auctionInfoData` is sorted, but can lead to many issues in the future if logic changes.
Context: [NextGenRandomizerNXT::calculateTokenHash](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/hardhat/smart-contracts/AuctionDemo.sol#L91-L92)

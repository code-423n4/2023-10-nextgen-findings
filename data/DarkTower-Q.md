## Summary

### Low Risk Issues
|Tag |Issue|Contexts|
|-|:-|:-:|
|LOW-1| Function Admin centralization risk in `emergencyWithdraw` function |1| 

Total: 1 contexts over 1 issues

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
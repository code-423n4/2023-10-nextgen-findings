## Summary

### Low Risk Issues
|Tag |Issue|Contexts|
|-|:-|:-:|
|LOW-1|Centralization Risks in `emergencyWithdraw` function|1| 

Total: 1 contexts over 1 issues

### LOW-1: Centralization Risks in `emergencyWithdraw` function
The function level admin can withdraw the funds using the function `emergencyWithdraw`. 

Context: [MinterContract::emergencyWithdraw](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L461)
```solidity
        function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
              uint balance = address(this).balance;
              address admin = adminsContract.owner();
              (bool success, ) = payable(admin).call{value: balance}("");
              emit Withdraw(msg.sender, success, balance);
    }
```
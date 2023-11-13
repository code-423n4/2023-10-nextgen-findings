## Summary

### Low Risk Issues
|Tag |Issue|Context|
|-|:-|:-:|
|Low-01| Centralization Risk for `emergencyWithdraw` function|1| 

Total: 1 context over 1 issue

### LOW-1: Centralization Risk for `emergencyWithdraw` function
A malicious function admin can arbitrarily choose to run `emergencyWithdraw` whenever there is any amount of fund in the MinterContract disrupting the normal flow of funds. If funds get withdrawn any and every time it would be an issue because then the state of collection total for NFT mints would be audio funds when there really isn't any fund in the contract. The admin contract owner will have some work on their hands to sift through a lot of state balances prior to the function admin withdrawing so they can reimburse each artists for instance correctly. It's a lot of work if the contract holds funds for a ton of collections and will now have to replenish each.
 
Context: [MinterContract::emergencyWithdraw](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/MinterContract.sol#L462)
```javascript
        @> function emergencyWithdraw() public FunctionAdminRequired(this.emergencyWithdraw.selector) {
              uint balance = address(this).balance;
              address admin = adminsContract.owner();
              (bool success, ) = payable(admin).call{value: balance}("");
              emit Withdraw(msg.sender, success, balance);
    }
```
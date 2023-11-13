[L-01] This implementation of isAdminContract returns true unconditionally. 

[Links to affected code](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/NextGenAdmins.sol#L83C1-L85C6)
> As seen in in [NextGenAdmins.sol](https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenAdmins.sol)`, the function below
```
    function isAdminContract() external view returns (bool) {
        return true;
    }
```
will always return true. 

> A call is made in [MinterContract.sol](https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/MinterContract.sol#L455) to `updateAdminContract`, where it is required that `INextGenAdmins(_newadminsContract).isAdminContract() == true`, this call will always return true, hence defeating the check becomes redundant. If this check will not be updated to check if the caller is an `adminContract`, then it should be removed completely to conserve gas.

If the modifier, `FunctionAdminRequired` passes, then the second check will always pass. Hence, all address that pass the first check are considered as Admins, if they are admins, then there's no need for the second check.
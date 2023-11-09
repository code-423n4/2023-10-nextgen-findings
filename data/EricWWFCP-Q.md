Refactor/Rethink Access Control
===============================

Essentially the Access Control methodology can be simplified and tightened up.

The various admin roles are trusted wallets so we have a low chance of malicious intent. Therefore will distill all access control related issues into one finding highlighting what I think are the important ones.

1) You are already using OpenZeppelin's Ownable contract. You may as well use their [AccessControl](https://docs.openzeppelin.com/contracts/5.x/api/access#AccessControl) contract instead. This will give the same functionality of NextGenAdmins in terms of managing specific Roles. It is a pre established, well audited solution that is easy to set up and will keep your code clean.

2) We have 4 trusted admin roles Owner, Admins, Collection Admins and Function Admins. I will assume though that there **is** a reason for the separation of roles otherwise we would only have one role. There might be some undesirable cases:
a. An Owner is an admin be default but it is possible for an Owner to not be an admin if they call `registerAdmin()` and set their own address to 'false'.
b. An old Owner can also remain an Admin inadvertently if they transfer or renounce their ownership. But they were added as an Admin upon contract creation and will still be an admin if `registerAdmin()` is not called again to revoke this privelege
c. An Owner can renounce their membership and then there is no more Owner and can never be one. In which case we can no longer register/deregister any more Admins

3) FunctionAdmins can steal all royalties. They can call `updateAdminContract()' with an arbitrary contract and then call emergencyWithdrawal() to send all the royalties to themselves.

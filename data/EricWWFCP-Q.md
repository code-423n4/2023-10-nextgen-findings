1. ***Refactor/Rethink Access Control***
Essentially the Access Control methodology can be simplified and tightened up.The various admin roles are trusted wallets so we have a low chance of malicious intent. Therefore will distill all access control related issues into one finding highlighting what I think are the important ones:

   a. You are already using OpenZeppelin's Ownable contract. You may as well use their [AccessControl](https://docs.openzeppelin.com/contracts/5.x/api/access#AccessControl) contract instead. This will give the same functionality of NextGenAdmins in terms of managing specific Roles. It is a pre established, well audited solution that is easy to set up and will keep your code clean.

   b. We have 4 trusted admin roles Owner, Admins, Collection Admins and Function Admins. I will assume though that there **is** a reason for the separation of roles otherwise we would only have one role. There might be some undesirable cases:
   - An Owner is an admin be default but it is possible for an Owner to not be an admin if they call `registerAdmin()` and set their own address to 'false'.
   - An old Owner can also remain an Admin inadvertently if they transfer or renounce their ownership. But they were added as an Admin upon contract creation and will still be an admin if `registerAdmin()` is not called again to revoke this privelege
   - An Owner can renounce their membership and then there is no more Owner and can never be one. In which case we can no longer register/deregister any more Admins

   c. FunctionAdmins can steal all royalties. They can call `updateAdminContract()' with an arbitrary contract and then call emergencyWithdrawal() to send all the royalties to themselves.

   d. Redundant modifiers are can be broken during contract updates. `FunctionAdminRequired()` and `CollectionAdminRequired` are both defined in NextGenCore AND MinterContract. These should be extracted out for two reasons:
   - Cleaner code to define it in one place and import it into each contract. One way to do this is to implement OZ's AccessControl contract like mentioned in finding (a.)
   - Keep modifier logic consistent. One of these contracts can possibly have a different definition of one of these modifiers than the other contract. This can happen if somebody calls `updateCoreContract()` and the "new" CoreContract has updated buggy code.

2. ***Suggestion for `payArtist()` function***
Vulnerabilities aside it might make sense to remove this logic altogether and just withdraw any royalties to a payment splitter contract. You can write your own or use an established, update-able, well audited solution that also simplifies your code. OpenZeppelin has a [PaymentSplitter](https://docs.openzeppelin.com/contracts/2.x/api/payment) contract which is discontinued but can still work or serve as an example and another popular solution is [0xSplits](https://splits.org/).

    Also, your current implementation assumes three "artist" addresses and two "team" addresses. With a payment splitter we can add/remove payees and update payout percentages if you ever decide you want a different configuration.
## Front-running freezeCollection operation

### Severity: LOW

### Code snippets
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L238-L253
https://github.com/code-423n4/2023-10-nextgen/blob/main/smart-contracts/NextGenCore.sol#L292-L295

### Description
Even though Collection Admin and Function Admin are trusted roles, a Collection admin who has lower privileges should not be able to act harmfully by using the Function admin's activity. Currently, collection admin has the ability to front-run function admin operation and in the case of the `freezeCollection` function, this commits irreversible changes.

### Proof of Concept
1. Function admin calls freezeCollection()
2. Collection admin front-run and update collection through `updateCollectionInfo`.
3. The updated collection data is frozen and the process cannot be reversed

### Recommendation
Enforce the usage of a private mempool service for all admin actions
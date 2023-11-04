# [Low-01] Array is push()ed but not pop()ed
Array entries are added but are never removed. Consider whether this should be the case, or whether there should be a maximum, or whether old entries should be removed. Cases where there are specific potential problems will be flagged separately under a different issue.
line: https://github.com/code-423n4/2023-10-nextgen/blob/8b518196629faa37eae39736837b24926fd3c07c/smart-contracts/AuctionDemo.sol#L60

# [Low-02] Immutable should be in UPPER_CASE
It is a good practice to write immutable and constants in upper case.